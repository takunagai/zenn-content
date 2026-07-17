---
title: "PhpStorm MCP サーバー の概要と Serena との比較と使い分け"
emoji: "🐸"
type: "tech"
topics: ["phpstorm", "mcp", "claudecode", "php", "serena"]
published: true
---

PhpStorm 2025.2 から、IDE そのものが MCP サーバーになりました。Claude Code などの AI エージェントが、PhpStorm の頭脳 ─ PHP 解析・インスペクション・安全なリネーム・実行・デバッグ ─ を直接叩けます。

一方 Serena は、言語サーバー（LSP）ベースの軽量でエディタ非依存な MCP サーバーです。同じ「MCP サーバー」でも守備範囲がまるで違います。

この記事では、PhpStorm MCP Server の概要と接続方法を整理し、Serena と比較して、どちらをいつ使うかまで落とし込みます。

## 先に結論

- **PhpStorm MCP Server** ＝ 商用 IDE の解析エンジンをエージェントに開放する仕組み。PHP の実務に最も強い。IDE 起動とライセンスが要る
- **Serena** ＝ LSP ベースでシンボル操作のプリミティブを配る独立サーバー。軽くて多言語、IDE 不要、OSS
- 併用できる。普段は Serena、PhpStorm を開く PHP 案件ではそちらの MCP も足す、が現実解

Serena 側の詳細は別記事にまとめています。あわせてどうぞ。
https://zenn.dev/takna/articles/serena-mcp-getting-started

## 最初に整理 ─ 「PhpStorm × MCP」には紛らわしい2つがある

ここを混同すると設定で迷います。PhpStorm まわりには、向きの違う 2 つの MCP の仕組みがあります。

| | 何をするか | 向き | 主な用途 |
| --- | --- | --- | --- |
| **PhpStorm MCP Server** | IDE のツール群を外部エージェントに開放する | エージェント → IDE（エージェントが IDE を操作） | コード解析・編集・実行・デバッグを任せる |
| **Claude Code 用 JetBrains プラグイン（Beta）の `ide`** | 診断・選択範囲・差分を Claude に渡す | IDE → Claude（Claude が情報を受け取る） | エラーの共有・diff 表示・選択箇所の文脈共有 |

本記事が主に扱うのは前者（PhpStorm MCP Server）です。後者の `ide` は Claude Code 側が持つ別機能で、モデルに公開されるツールは診断取得（read-only）が中心です。名前がどちらも MCP なので、設定画面を触るときは「どちらの話か」を意識してください。

## PhpStorm MCP Server の概要

### いつ入った機能か

PhpStorm **2025.2** で内蔵されました。それ以前は `@jetbrains/mcp-proxy`（npx で起動する外部プロキシ）を使う方式でしたが、これは 2025.2 で IDE に統合され、公式に非推奨となっています。今から使うなら内蔵版一択です。

### 何を開放できるか

外部エージェントに公開されるツールは `Settings | Tools | MCP Server | Exposed Tools` で確認・取捨選択できます。カテゴリは次のように広範です。

- ファイル操作・読み取り・テキスト編集
- 検索（テキスト / 正規表現 / 構造検索）
- コード解析・インスペクション・quick-fix
- リファクタリング（参照込みのリネームなど）
- 実行（run configuration）・ターミナル・ビルド
- PHP デバッガ（Xdebug）
- PHP プロジェクト文脈（Composer・PHP 設定・モジュール）
- VCS・IDE アクション・データベース

要は「PhpStorm が普段やっていること」をそのままエージェントに委ねられる、という理解でよいです。

## セットアップ ─ Claude Code から接続する

### 手順

1. `設定 | ツール | MCP サーバー` を開き、"MCP サーバーの有効化" にチェック
2. 同じ画面の **クライアントの自動構成** から、接続したいクライアント（Claude Code / Claude Desktop / Cursor / VS Code など）を選ぶ。対応する設定ファイルが自動で書き込まれる
3. 手動で設定したい場合は、同画面の "SSE 構成のコピー / Stdio 構成のコピー / HTTP ストリーム構成のコピー" から接続方式に応じた設定をコピーして使う

接続の具体的なコマンド（`claude mcp add ...` の文字列）や生成される JSON・ポート番号は、現時点の公式ドキュメント本文には明記されていません。**Auto-Configure に任せるのが正攻法**で、手順どおりならコマンドを手書きする必要はありません。

:::message alert
MCP サーバー の**プラグイン（コンポーネント）自体は既定で有効**ですが、外部クライアントと通信する**サーバー機能のほうは既定で無効**です（JetBrains 公式ブログで明言）。バージョンによらず、接続前に必ず `設定 | ツール | MCP サーバー` で `MCP サーバーの有効化` をクリックしてください。
:::

### PHP 特化のスキルを足す（任意）

[phpstorm-claude-marketplace](https://github.com/JetBrains/phpstorm-claude-marketplace/blob/main/README.md) プラグインを入れると、PHP 向けのスキルとフックが加わります。

- スキル: "PHP Code Review" / "PHP Project Guide" / "Upgrade PHP" など、文脈依存の作業テンプレート
- フック: コード編集後に PhpStorm のインスペクションを自動実行し、その結果を Claude にフィードバックする

設定は `設定 | ツール | PHP Claude Skills` から行います。導入には MCP Server の有効化と Claude Code CLI が前提です。

## Serena のおさらい

Serena は Python 製の独立 MCP サーバーです。現在の公式推奨は `uv tool install -p 3.13 serena-agent` でインストールし、`serena start-mcp-server` で起動する方式です。内部で LSP を起動し、コードを「シンボルの木」として扱い、`find_symbol`（定義探索）・`find_referencing_symbols`（参照追跡）・`replace_symbol_body`（シンボル単位の編集）・プロジェクトメモリといったツールを配ります。既定では IDE を必要とせず、LSP が対応する言語ならどれでも使えます（近年は JetBrains をバックエンドにするオプションも加わりましたが、本記事は既定の LSP 運用を前提とします）。

## 比較表

| 観点 | Serena | PhpStorm MCP Server |
| --- | --- | --- |
| 正体 | 独立プロセス（Python、uv でインストール） | IDE 本体が MCP サーバー化 |
| 前提 | なし（uv でインストールして起動） | PhpStorm 起動 ＋ ライセンス（有料） |
| 解析基盤 | LSP（言語サーバー） | PhpStorm の解析エンジン |
| 対応言語 | LSP 対応の全言語 | JetBrains 対応言語（PHP に最強） |
| コード検索 | シンボル検索・参照追跡 | シンボル ＋ テキスト / 正規表現 / 構造検索 |
| リファクタ | シンボル本体の差し替え | 参照込みのリネーム等 |
| インスペクション | LSP 診断のみ | PhpStorm のインスペクション・quick-fix |
| 実行・ビルド | シェル実行 | run config / ターミナル / ビルド |
| デバッグ | なし | Xdebug でのステップ実行 |
| VCS・依存 | なし | リポジトリ情報・Composer・PHP 設定 |
| メモリ | あり（`.serena/memories/`） | なし（IDE インデックスで代替） |
| 接続 | stdio | stdio / SSE / HTTP Stream（自動設定） |
| 動作の重さ | 軽い | 重い（フル IDE 常駐） |
| コスト | 無料・OSS | 有料・クローズド |

## 使い分け

**PhpStorm MCP Server が向くケース**

- PHP や WordPress の実務で、すでに PhpStorm を開いている
- エージェントに、実際の PHP 解析・安全なリネーム・Xdebug デバッグ・run/build まで任せたい
- IDE の重さとライセンスを許容できる

**Serena が向くケース**

- Zed / Neovim / CLI など、エディタを問わず軽く使いたい
- 多言語のプロジェクトで、シンボルナビとシンボル単位編集、プロジェクトメモリがあれば足りる
- ライセンス不要・OSS で完結させたい

**併用する**

どちらも MCP サーバーとして同時に登録できます。普段は軽い Serena、PhpStorm を立ち上げる PHP 案件の日はそちらの MCP も足す、という併用が無理のない落としどころです。

## セキュリティの注意

- PhpStorm MCP Server には **brave mode** があります。確認なしでシェルやランを実行できてしまうため、エージェント運用では無効のままが安全です
- Claude Code 用プラグインの `ide` 接続は、既定では loopback（`127.0.0.1`）のみです。ネットワーク公開の設定を有効にすると通信が平文で流れる点に注意してください

## まとめ

- PhpStorm 2025.2 から IDE が MCP サーバーになり、PHP 解析・リファクタ・実行・デバッグをエージェントに開放できる
- 「PhpStorm MCP サーバー（IDE を操作させる）」と「Claude Code の `ide`（情報を受け取る）」は向きが逆。混同しない
- 接続は `設定 | ツール | MCP サーバー` の有効化 ＋ 自動構成。コマンドの手書きは不要
- 軽く多言語の Serena と、PHP に深い PhpStorm MCP は競合ではなく補完。案件に応じて使い分け・併用する
