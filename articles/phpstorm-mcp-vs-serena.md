---
title: "PhpStorm MCP サーバーの概要と Serena との比較と使い分け"
emoji: "🐸"
type: "tech"
topics: ["phpstorm", "mcp", "claudecode", "php", "serena"]
published: true
---

PhpStorm 2025.2 から、IDE そのものが MCP サーバーになりました。Claude Code などの AI エージェントが、PhpStorm の頭脳 ─ PHP 解析・インスペクション・安全なリネーム・実行・デバッグ ─ を直接叩けます。

一方 Serena は、言語サーバー（LSP）ベースの軽量でエディタ非依存な MCP サーバーです。同じ「MCP サーバー」でも守備範囲がまるで違います。

この記事では、PhpStorm MCP Server の概要と接続方法を整理し、Serena と比較して、どちらをいつ使うかまで落とし込みます。

## 先にまとめ

- **PhpStorm MCP Server** ＝ 商用 IDE の解析エンジンをエージェントに開放する仕組み。PHP の実務に最も強い。IDE 起動とライセンスが要る
- **Serena** ＝ LSP ベースでシンボル操作のプリミティブを配る独立サーバー。軽くて多言語、IDE 不要、OSS
- 同じプロジェクトで両方を常時 ON にはしない。PhpStorm を開く PHP 案件は PhpStorm MCP、それ以外は Serena と、プロジェクト単位で片方に寄せるのが基本

Serena 側の詳細は別記事にまとめています。あわせてどうぞ。
https://zenn.dev/takna/articles/serena-mcp-getting-started

## PhpStorm の MCP は2つあり紛らわしい

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

各カテゴリの詳しいツール一覧は、JetBrains 公式ドキュメントにまとまっています ─ [MCP Server | JetBrains IDE Documentation](https://www.jetbrains.com/help/idea/mcp-server.html)。

## WordPress 開発で何が便利か

WordPress テーマ開発の文脈では、とくにデータベースを直接読める点と、実行時の状態をデバッガで覗ける点が調査を速くし、デプロイまで IDE 経由で完結できるのも往復の手間を減らします。まずはローカルのデータベース接続を登録しておくと、「表示がおかしい」の切り分けが一段軽くなります。ここでは子テーマ中心・ビルド工程なしの WordPress テーマ開発を前提に、実際に効いた順で紹介します。

### 1. データベース操作

WordPress は設定も本文もデータベースに載るため、ここに直接触れられるかどうかが調査速度を大きく変えます。

- 主なツール: `create_database_connection` / `execute_sql_query` / `preview_table_data` / `introspect_schema`
- 使いどころ: `wp_posts` の `post_content`（ブロックエディタのブロック設定はここに JSON として入ります）、`wp_options`、`postmeta`（ACF のカスタムフィールド）、カスタム投稿タイプのデータを直接読めます。とくに「テーマのコードは正しいのに表示が崩れる」ケースは、原因がデータベース側のブロック設定であることが多く、SQL で本文を覗けば一撃で切り分けられます
- `wp-cli` がホストの PATH に無い環境でも、IDE 経由でデータベースを読めるのが実用的です

注意点として、データベースへの書き込み（データやスキーマの変更）は影響が大きいため、基本は参照にとどめ、変更が必要なときは必ず内容を確認してから実行します。また、本番のレンタルサーバーのデータベースは外部から直接つなげないことが多いので、この操作はローカル環境が主戦場になります。

### 2. リモートデプロイ（FTP アップロード）

PhpStorm に登録済みの転送設定（FTP / SFTP）を使って、ファイルを本番サーバーへアップロードできます。手動で FTP クライアントを開いたり、FTPを繋いで操作するスキルを作成しなくても、AI エージェントからデプロイまで完結します。

- 主なツール: `open_file_in_editor` で対象を開く → `invoke_ide_action`（アクション ID `PublishGroup.Upload`）
- 使いどころ: テーマの CSS や `functions.php` を修正したあと、そのまま本番へ反映する一連の流れを自動化できます

つまずきやすい点を先に挙げておきます。

- 転送先を「デフォルトサーバー」に設定していないと、アップロードのアクションが「現在のコンテキストでは利用できません」と弾かれます
- 初回は「アップロードの確認」ダイアログや FTP パスワードの入力で処理が止まり、AI 側の呼び出しがタイムアウトすることがあります。これは失敗ではなく、IDE 側で応答を待っている状態です。確認ダイアログの「今後このメッセージを表示しない」にチェックを入れ、パスワードを保存すれば、以降は無言で通ります
- 「変更を保存するたびに自動アップロード」するトグルも用意されていますが、検証前の変更まで本番へ飛ぶため、有効化はおすすめしません

デプロイ後は必ず、公開 URL のファイルを実測して反映を確認します。`curl` でファイルの `content-length` と `Last-Modified`、キャッシュ回避のためにバンプした `?ver=` の値が更新されているかを見ます。多くの構成では静的ファイルはディスクの実体がそのまま返るため、`curl` の結果がサーバー側の真実を映します。ブラウザ側の長期キャッシュ（`max-age` が数か月に設定されている場合など）は、まっさらな `curl` には影響しません。この切り分けを押さえておくと、「反映されない」の原因がサーバー未反映なのか、ブラウザキャッシュなのかで迷わなくなります。

### 3. Xdebug ステップデバッグ

ブレークポイントを張って処理を止め、その瞬間の変数を覗くステップデバッグも、MCP から制御できます。

- 主なツール: `set_breakpoint` → `start_debugger_session` → `get_frame_values` / `evaluate_expression` / `run_to_line`
- 使いどころ: フックがどの順で発火しているか、`$wp_query` に何が入っているか、テンプレートに渡る変数の実際の中身は何か ─ 実行時の状態を止めて確認できます。`var_dump` を仕込んでは消す作業から解放されます

ローカル開発環境には Xdebug が同梱されていることが多いですが、利用には有効化とリッスンの設定が必要です。

### 4. コード品質チェック（インスペクション）

PhpStorm のインスペクションは、コマンドラインの静的解析より広い範囲を見てくれます。

- 主なツール: `get_file_problems` / `lint_files` / `get_inspections` / `apply_quick_fix` / `reformat_file`
- 使いどころ: 未定義関数の呼び出し、型の不整合、コーディング規約違反などを、ファイル単位でまとめて拾えます。クイックフィックスやフォーマットの適用も IDE の作法に沿って行えます

### 5. セマンティック検索とリファクタリング

テキストの grep ではなく、コードの意味を理解した検索・置換ができます。

- 主なツール: `analyze_calls`（コールヒエラルキー ─ この関数を呼んでいるのは誰か）/ `search_symbol` / `get_symbol_info` / `search_structural` / `rename_refactoring`（プロジェクト横断の安全なリネーム）
- 使いどころ: 親テーマが大量に用意しているフックを追いかけるとき、「この関数はどこから呼ばれているか」を正確にたどれます。リネームも参照ごと安全に書き換えられます

### 6. IDE 操作全般

`invoke_ide_action` と `search_ide_actions` を使えば、IDE のほぼあらゆるアクションを呼び出せます。先ほどのアップロードもこの仕組みです。

- 使いどころ: インポートの最適化、プロジェクトのビルド（`build_project`）、実行構成の起動（`execute_run_configuration`）など。GUI のメニューから叩く操作を、そのままエージェントから実行できます

### メモ

- Git 操作系のツールもありますが、`git` / `gh` の CLI で運用している場合は、そちらのほうが小回りが利きます
- WordPress では使いませんが、Laravel 向けの専用ツール群も用意されています

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

### WordPress 案件は規約が変わる（WPCS）

一点、注意があります。上の "PHP Project Guide" を含む JetBrains の PHP スキルは、フレームワークが Laravel / Symfony のときだけその流儀に合わせ、それ以外はすべて汎用 PHP（PSR/PER）として案内します。ところが WordPress の規約は WPCS（WordPress Coding Standards）─ タブインデント・snake_case・Yoda 条件・`global $wpdb` ─ で、PSR/PER と真っ向から衝突します。WordPress 案件でそのまま従うと、かえって規約違反のコードになります。

そこで、この穴を埋める自作スキルを公開しています。WPCS 準拠のコード生成・レビュー指針に加えて、WPCS 3.x + PHP_CodeSniffer と PHPStan（phpstan-wordpress）の導入、PhpStorm 側の設定ナビまでを一括で行うものです。

- スキル: https://github.com/takunagai/agent-assets/tree/main/skills/wordpress-project-guide
- ドキュメント: https://github.com/takunagai/agent-assets/blob/main/docs/wordpress-project-guide.md

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

**同じプロジェクトでは片方に絞る**

技術的には同時に登録できますが、同じプロジェクトでの常時併用はおすすめしません。機能が大きく重複するうえ（シンボル検索・参照追跡・リネーム）、MCP のツールは呼び出しごとにコンテキストへ読み込まれ、数が増えるほど正しいツールを選ぶ精度が落ちるためです。PhpStorm MCP だけで数十ツールあり、そこへ Serena を重ねると取り違えとトークンの浪費を招きます。

指針はシンプルです ─ プロジェクト単位でどちらか一方に。PhpStorm で PHP を触るなら PhpStorm MCP、それ以外は Serena。両方登録するなら、Serena を local スコープに限定し、使わない側は無効化しておきます。

## セキュリティの注意

- PhpStorm MCP Server には **brave mode** があります。確認なしでシェルやランを実行できてしまうため、エージェント運用では無効のままが安全です
- Claude Code 用プラグインの `ide` 接続は、既定では loopback（`127.0.0.1`）のみです。ネットワーク公開の設定を有効にすると通信が平文で流れる点に注意してください

## まとめ

- PhpStorm 2025.2 から IDE が MCP サーバーになり、PHP 解析・リファクタ・実行・デバッグをエージェントに開放できる
- 「PhpStorm MCP Server（IDE を操作させる）」と「Claude Code の `ide`（情報を受け取る）」は向きが逆。混同しない
- 接続は `設定 | ツール | MCP サーバー` の有効化 ＋ 自動構成。コマンドの手書きは不要
- 軽く多言語の Serena と、PHP に深い PhpStorm MCP は競合ではなく補完。ただし同じプロジェクトでは片方に絞る（ツールの重複と氾濫を避けるため）
