---
title: "Serena 入門 ─ Claude Code に正確なコード理解を足す MCP サーバー"
emoji: "🧭"
type: "tech"
topics: ["claudecode", "mcp", "serena", "ai", "typescript"]
published: true
---

![Claude が虫眼鏡で多数の似たコード片から目的のシンボルを 1 つだけ正確に照らし出すエディトリアル風イラスト](/images/serena-mcp-getting-started/eyecatch.png)

いまさら感がありますが、Serena MCP について解説します。

Serena は、Claude Code などの AI コーディングエージェントに「コードを構文レベルで理解する目」を与える MCP サーバーです。grep が文字の並びを探すのに対し、Serena は言語サーバー（LSP）を通してシンボル（関数・クラス・変数）の定義や参照を正確にたどります。

この記事のとおりに進めれば、導入からよく使う操作まで一通り再現できます。つまずきどころもメモしています。検証環境は Mac + Claude Code + TypeScript プロジェクトですが、他の言語やエージェントでも大筋は同じです。

## この記事で分かること

- Serena が何者で、何が嬉しいか
- Claude Code への導入（コマンド1行）
- よく使う基本ツールの使い方
- 実際に踏んだつまずき3つと回避法

## Serena とは

Serena は MCP（Model Context Protocol）サーバーの一つです。内部で LSP を起動し、ソースコードを「シンボルの木」として扱います。その上でエージェントに、シンボル単位の検索・参照追跡・編集といったツール群を提供します。

### grep と何が違うか

| 観点 | grep / 文字列検索 | Serena |
| --- | --- | --- |
| 検索の単位 | 文字の並び | シンボル（関数 / クラス / 変数） |
| 「定義だけ」を抽出 | 難しい（同名の文字列も拾う） | できる |
| 呼び出し元の列挙 | 手作業で絞り込み | 参照追跡で正確に |
| 編集の単位 | 行・文字範囲 | シンボル本体を丸ごと差し替え |
| トークン消費 | 該当行を広めに読む | 必要なシンボルだけ読むので節約 |

まとめると、Serena の利点は次の3点です。

1. 定義や参照を「意味」で正確にたどれる（同名の別物を拾わない）
2. シンボル単位で編集できるので、行番号ズレの事故が起きにくい
3. 必要な箇所だけ読むため、コンテキスト（トークン）を節約できる

## こんなときに導入する

Serena は万能ではありません。向き・不向きを先に押さえておくと、無駄な導入を避けられます。

導入する価値が高いケース。

- 中規模以上のコードベースで、シンボルの定義や参照を頻繁にたどる
- リファクタリング・リネーム・影響範囲の調査が多い
- LSP が効く静的型付き言語（TypeScript、Python、Go、Java など）
- 大きいファイルを丸ごと読ませず、トークンを節約したい

無理に入れなくてよいケース。

- 小規模・単一ファイル中心で、標準の grep や Read で足りる
- 主に文章・設定・Markdown を扱い、コードのシンボルが薄い
- LSP のサポートが弱い、または未対応の言語

迷ったら「grep でシンボルを追うのがつらいと感じ始めたら入れる」くらいの温度感で十分です。

## 前提ツール

- uv（uvx）: Serena は Python 製で、uvx により隔離環境で実行します
- Claude Code: 導入済みを前提とします

uv が未導入なら入れておきます。

```bash
brew install uv
```

確認します。

```bash
uv --version
claude --version
```

## 導入 ─ 3ステップ

### 1. 対象プロジェクトへ移動

```bash
cd /path/to/your-project
```

### 2. Serena を登録する

プロジェクト内で、local スコープに登録します（`claude mcp add` の既定スコープが local です）。

```bash
claude mcp add serena -- \
  uvx --from git+https://github.com/oraios/serena \
  serena start-mcp-server --context claude-code --project "$(pwd)"
```

- `--context claude-code` はコーディング支援向けの設定です（旧名 `ide-assistant` も動きますが、非推奨の警告ログが出るため新しい記事では `claude-code` を使います）
- `--project "$(pwd)"` で対象プロジェクトを固定します（`$(pwd)` は実行時に絶対パスへ展開されます）

### 3. 接続を確認する

```bash
claude mcp list
```

`serena: ... ✔ Connected` と表示されれば成功です。初回はブラウザに Serena のダッシュボード（`http://localhost:24282/dashboard/`）が開き、ツール呼び出しのログを確認できます。

> [!note] 起動時に開くページは閉じてよい
> Claude Code 起動のたびにブラウザで開くのは Serena の**ダッシュボード**（ログ確認用の画面）です。Serena 本体は Claude Code が起動したプロセスで動いており、ブラウザとは独立しています。タブを閉じても機能は止まりません。キープしておく必要はありません。
> 毎回開くのを止めたいときは `~/.serena/serena_config.yml` で `web_dashboard_open_on_launch: false`、画面自体が不要なら `web_dashboard: false` に設定します（閉じても、無効化しても Serena の動作には影響しません）。

登録後、Serena はプロジェクト直下に `.serena/` を作成します。`.serena/cache` は生成物なので Git 管理の対象外にします（Serena が `.serena/.gitignore` を用意します）。

> [!note] グローバル登録に注意
> user スコープで `--project /特定パス` を固定して登録すると、別のプロジェクトを開いても前のパスを掴み続けます。登録は必ずプロジェクト内で local スコープに行ってください。

## 事前インデックス（任意・高速化）

シンボル検索を速くするため、最初に索引を作っておきます。

```bash
uvx --from git+https://github.com/oraios/serena serena project index
```

`Indexed files per language: typescript=486` のように表示されれば完了です。以後の検索は索引ヒットで速くなります。規模の大きいプロジェクトほど効果が出ます。

> [!note] 索引はときどき作り直す
> 索引はある時点のスナップショットです。日常の小さな編集では作り直す必要はなく、索引に無いシンボルは言語サーバーがその場で拾うため、多少古くても壊れません。ただし次のような場面では `serena project index` を叩き直すと、検索の取りこぼしや速度低下を防げます。
>
> - 大規模なリファクタリングのあと
> - ファイルの大量な追加・移動・リネームのあと
> - ブランチを切り替えてコード構成が大きく変わったあと
> - シンボル検索が遅い、または見つからないと感じたとき
>
> 索引は毎回まるごと上書きで再生成されるので、いつ実行しても安全です。

## 使ってみる

ポイントは、自分でツールを直接叩くのではなく、Claude に日本語で頼むと裏で Serena のツールが走るという点です。以下は Claude への依頼文の例です。

### まずファイルの地図を見る

> `src/hooks/use-form-submission.ts` の全体像を教えて

裏で `get_symbols_overview` が走り、ファイル内のシンボル一覧が返ります。新しいファイルを読むときの最初の一手です。

### シンボルを探す ─ find_symbol

> useFormSubmission がどこにあるか探して

裏で `find_symbol` が走り、定義位置（ファイル: 行）が返ります。

精度を上げたいときは「名前パス」で狙えます。

| 形式 | 例 | 意味 |
| --- | --- | --- |
| 単純名 | `useFormSubmission` | 同名のシンボルすべて |
| 相対パス | `useFormSubmission/handleSubmit` | 「親 / 子」のサフィックス一致 |
| 絶対パス | `/useFormSubmission/handleSubmit` | ファイル内フルパスの完全一致 |

よく使うオプションは次のとおりです。

- `relative_path`: 探索範囲をファイルやディレクトリに絞る
- `depth: 1`: クラスの子（メソッド一覧）まで取得する
- `include_body: true`: シンボルの本体ソースを取得する（読みたいときだけ）

### 呼び出し元を洗う ─ find_referencing_symbols

> useFormSubmission を呼んでいる箇所を全部挙げて

裏で `find_referencing_symbols` が走り、呼び出し元を正確に列挙します。リネームや影響範囲の調査の前に使うと安全です。

### シンボル単位で編集する ─ replace_symbol_body

> useHeroHeight の中身を（…指示…）のように書き換えて

関数やクラスを本体ごと差し替えます。行範囲を指定する編集と違い、行番号のズレによる事故が起きにくいのが利点です。

### プロジェクトの記憶 ─ メモリ

Serena はプロジェクトの知識を `.serena/memories/` に保存できます。

> このプロジェクトの構成をメモリに残しておいて

次回以降、`read_memory` で呼び出して文脈を素早く共有できます。

## つまずきポイント3つ

実際に踏んだものです。

### 1. 重複登録で起動時に Failed to connect

登録をやり直すと、`serena` と `serena-mcp-server`（旧エントリポイント）が二重に残ることがあります。`claude mcp list` で ✘ が出ていたら、余分なほうを削除します。

### 2. `remove -s local` が「No MCP server」で失敗する

`claude mcp get <name>` は「`-s local` で消せ」と案内しますが、その通り実行すると失敗する場合があります。スコープ指定を外すと消せます。

```bash
claude mcp remove serena-mcp-server
```

これは Claude Code 側の既知の挙動で、バージョンによって変わる可能性があります。

### 3. グローバル登録がプロジェクトを漏らす

前述のとおり、user スコープに `--project` を固定して登録すると別プロジェクトへ漏れます。登録はプロジェクト内で local スコープに統一します。

## まとめ

- Serena はコードを構文で理解する MCP サーバー。grep より正確で省トークン
- 導入は `claude mcp add` での登録と `claude mcp list` での接続確認、`serena project index` の索引作成
- 基本の流れは overview → find_symbol → find_referencing_symbols → replace_symbol_body
- つまずきは「重複登録」「remove のスコープ」「グローバル登録」の3つ
