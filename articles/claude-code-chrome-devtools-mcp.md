---
title: Claude Code に Chrome DevTools for Agents（chrome-devtools-mcp）を繋ぐ実践ガイド
emoji: 🐸
type: tech
topics:
  - claudecode
  - chrome
  - devtools
  - mcp
published: true
---

![Claude Code と chrome-devtools-mcp を繋ぐ](/images/claude-code-chrome-devtools-mcp/eyecatch.png)

**Chrome DevTools for Agents v1** を使うと、Claude Code を Chrome の DevTools に MCP サーバー経由で接続し、Lighthouse 監査・デバイスエミュレーション・メモリリーク検出などをエージェントが直接実行できます。これまで手動でブラウザを操作して確認していた作業を、コードを書いたのと同じセッション内で完結できるようになります。

2026 年 5 月に安定版 v1.0 がリリースされ、Chrome 149（2026 年 6 月）でさらに機能が追加されました。この記事では Claude Code へのセットアップ手順と実践的なユースケースを解説します。

:::message
**「Chrome DevTools for Agents」と「chrome-devtools-mcp」は同じもの**

**Chrome DevTools for Agents** は Google Chrome チームによる機能の呼称（総称）で、その実体が `chrome-devtools-mcp` という npm パッケージ（MCP サーバー）です。Claude Code にインストール・設定するのはこの `chrome-devtools-mcp` です。本記事では、機能の総称として「DevTools for Agents」、コマンドや設定では `chrome-devtools-mcp` と表記します。
:::

:::message
**Claude in Chrome（Chrome 拡張機能）との違い**

Anthropic は Chrome 拡張機能 **Claude in Chrome** も提供しています（2025 年 11 月ベータ開始・有料プラン向け）。こちらはウェブページ上のクリック・入力・ナビゲーションを自動化する一般ユーザー向けの機能です。DevTools for Agents は開発者向けのデバッグ用途で、MCP サーバー経由で Lighthouse 監査やメモリ解析を行う点が異なります。「ブラウザで何かをする（do）」が前者、「ブラウザの内部を調べる（inspect）」が後者です。
:::

## DevTools for Agents (chrome-devtools-mcp) とは

Chrome DevTools for Agents は、AI コーディングエージェントとブラウザの DevTools を繋ぐ仕組みです。

従来、エージェントはコードを生成できても、そのコードが実際にブラウザ上でどう動くかは観察できませんでした。DevTools for Agents は MCP（Model Context Protocol）サーバーとして実装されており、エージェントに DevTools のデバッグ機能への直接アクセスを与えます。

エージェントへのインターフェースは 3 種類提供されています：

- **MCP サーバー**: LLM と DevTools のデバッグ機能を繋ぐ標準的な接続方法
- **CLI**: バッチ処理向けのトークン効率が高い代替手段
- **エージェントスキル**: アクセシビリティやパフォーマンスのデバッグ用の専門的な指示セット

Claude Code のような AI エージェントが「実際のユーザーと同じようにサイトを体験できる」という表現が公式ドキュメントにあります。コードを書いた直後に、同じセッション内でブラウザの挙動まで確認できる状態になります。

## Claude Code へのセットアップ

### 前提条件

- Claude Code（最新版）
- Google Chrome（最新版）

### インストール手順

Claude Code のスラッシュコマンドで 2 ステップで完了します。

まずマーケットプレイスレジストリを追加します：

```
/plugin marketplace add ChromeDevTools/chrome-devtools-mcp
```

次にプラグインをインストールします：

```
/plugin install chrome-devtools-mcp@chrome-devtools-plugins
```

以上でセットアップは完了です。

### 動作確認

Chrome で任意のページを開き、Claude Code で次のように確認します：

```
現在ブラウザで開いているページのタイトルとURLを取得してください
```

ページ情報が返ってくれば DevTools との連携が確立できています。

### 既存の Chrome をそのまま使う（二重起動を避ける）

デフォルトでは、MCP が呼ばれるたびに新しい Chrome ウィンドウが起動します。普段使いの Chrome をそのまま使い、余分なウィンドウを開かせたくない場合は、Chrome 144 以降の auto-connect を使います。

![chrome://inspect でリモートデバッグを有効化する](/images/claude-code-chrome-devtools-mcp/chrome-inspect-remote-debugging.png)

1. 普段使いの Chrome で `chrome://inspect/#remote-debugging` を開き、リモートデバッグを有効化します
2. `~/.claude.json` の `mcpServers` に `--autoConnect` 付きで `chrome-devtools` を登録します（プラグインと同じキー名にすると、この設定が使われます）：

```json
"mcpServers": {
  "chrome-devtools": {
    "command": "npx",
    "args": ["chrome-devtools-mcp@1.2.0", "--autoConnect"]
  }
}
```

3. Claude Code を再起動します。MCP がツール実行時に接続許可ダイアログを出すので、許可します

これで別の Chrome を立てず、ログイン済みの普段使いセッションにそのまま接続できます。`--remote-debugging-port` や専用プロファイルの用意は不要です。

:::message
auto-connect ではログイン済みのセッション（Cookie・アカウント情報を含む）がエージェントから見えます。信頼できる用途でのみ使ってください。
:::

## 実践ユースケース

### 1. Lighthouse 監査を自動化する

エージェントが Lighthouse 監査を実行し、アクセシビリティ、SEO、パフォーマンス、ベストプラクティスのスコアを取得できます。問題箇所の特定からコードへの反映まで、同じセッション内で完結します。

```
localhost:3000 を開いて Lighthouse 監査を実行し、アクセシビリティスコアと
SEO スコアを報告してください。スコアが 90 未満の項目がある場合は、
問題の詳細と修正方法を提示してください
```

従来は「DevTools を開く → Lighthouse パネルに移動 → 実行 → スコア確認 → コードエディタに戻る → 修正 → 再確認」という往復が必要でした。これが一つのプロンプトで完結します。

品質ゲートとしての使い方も有効です。デプロイ前のチェックとして組み込むことで、アクセシビリティや SEO の問題が本番環境に出る前に検出できます。公式ドキュメントでは「ランタイムを理解するリンター」と表現されています。

### 2. モバイル・低速回線をエミュレートする

実際のモバイル環境をシミュレートして問題を再現・修正できます。

```
iPhone 14 のデバイスサイズでこのページを確認し、ナビゲーションメニューが
正しく表示されるかテストしてください。その後、3G 相当の低速回線でリロードして
LCP（Largest Contentful Paint）の値を教えてください
```

エミュレーション対象：

| 項目 | 内容 |
|------|------|
| ウィンドウサイズ | デバイス別の解像度 |
| 位置情報 | GPS 座標のシミュレート |
| ネットワーク速度 | 3G、低速モバイルなど |
| CPU 速度 | 低スペックデバイスの再現 |

Chrome 149（v1.1.1）では、カスタム HTTP ヘッダーのエミュレーションも追加されました。認証トークンやカスタムユーザーエージェントを設定した状態でのデバッグに使えます。ステージング環境や A/B テストのフラグを持った状態の確認が、ブラウザの設定を手動変更せずに行えます。

### 3. 認証済みセッションをエージェントに引き継ぐ

これが DevTools for Agents の機能の中でも特に実用的な機能です。

通常、エージェントはサンドボックス化されたブラウザインスタンスを起動します。そのため、ログインが必要なページのデバッグでは、エージェントが改めて認証を行う必要がありました。

**auto-connect** 機能を使うと、現在開いているブラウザのセッションをそのままエージェントと共有できます。

```
現在ログイン済みの管理画面（http://localhost:3000/admin/dashboard）で、
データ読み込み時にコンソールに出力されているエラーを調査してください
```

認証情報を渡し直すことなく、ログイン後の画面のデバッグをエージェントに依頼できます。OAuth や SSO を使った環境でも有効で、「ログインしている状態で発生するバグ」の調査をエージェントに委任できます。

### 4. メモリリークを検出する

SPA（シングルページアプリケーション）のメモリリーク検出もエージェントに任せられます。

```
このページで、トップ→一覧→詳細→トップのナビゲーションを 3 回繰り返してから
ヒープスナップショットを取得し、切り離された DOM ノードやメモリリークの兆候を
報告してください
```

エージェントはヒープスナップショットを取得して比較し、メモリが増加し続けるパターンを特定します。問題箇所の特定とコードへの修正提案がセットで返ってきます。

手動でメモリプロファイリングを行うには、ナビゲーションの繰り返し操作、スナップショットの複数回取得、比較分析と知識が必要でした。これをプロンプト一つで依頼できます。

### 5. エージェントが発行する CDP コマンドを観察する

chrome-devtools-mcp は内部的に **Chrome DevTools Protocol（CDP）** を介してブラウザと通信しています。エージェントが `list_pages` を呼べば `Target.getTargets` が、`take_screenshot` を呼べば `Page.captureScreenshot` が CDP コマンドとしてブラウザに送られています。

:::message
**CDP（Chrome DevTools Protocol）とは**
Chrome がデバッグツールや自動化ツールと通信するためのプロトコルです。DevTools 自体が内部で使っているのと同じ低レベル API で、ページのナビゲーション・スクリーンショット取得・ネットワーク監視・JavaScript 実行などをプログラムから制御できます。Puppeteer や Playwright も同じプロトコルで Chrome を操作しています。
:::

**プロトコル モニター**を使うと、このやり取りをリアルタイムで可視化できます。

エージェントが期待通りに動かないとき——MCP ツールは呼ばれているはずなのにブラウザが反応しない、特定の操作で予期しないエラーが出る——といった状況で、CDP レベルまで原因を追跡できます。

**有効化手順：**

1. Chrome で DevTools を開く（`F12` または `Cmd+Option+I`）
2. 歯車アイコン（設定）→「試験運用版」タブ
3. 「プロトコル モニター」にチェックを入れて DevTools を再起動（再読み込みボタン）
4. DevTools 上部「その他のツール」→「プロトコル モニター」

![DevTools の試験運用版で Protocol Monitor を有効化する](/images/claude-code-chrome-devtools-mcp/devtools-experiments-protocol-monitor.png)

有効化すると、DevTools が送受信するすべての CDP リクエスト・レスポンスが一覧表示されます。「保存」ボタンで JSON 形式のエクスポートも可能で、デバッグセッション中の通信内容を後から解析できます。CDP コマンドを直接入力して送信するフィールドも備わっており、低レベルな動作を手動でテストすることもできます。

エージェントが Lighthouse 監査を実行した際に内部でどの CDP コマンドが走ったかを追えるため、カスタムツール開発時の動作確認にも使えます。

参考：[Chrome DevTools 92 の新機能 — プロトコル モニター](https://developer.chrome.com/blog/new-in-devtools-92?utm_source=devtools&utm_campaign=stable&hl=ja#protocol-monitor)

## Chrome 149（v1.1.1）で追加された機能

2026 年 6 月 2 日の Chrome 149 リリースで、DevTools for Agents に新たな機能が追加されました（現時点では試験運用段階）。

**カスタムのサードパーティ製ツール**

ウェブページが JavaScript を通じてカスタムデバッグツールを定義し、エージェントに公開できるようになりました。React や Vue のコンポーネントデバッガーなど、既存のデバッグツールをエージェントから操作できるようになることを想定した機能です。

利用するには `chrome-devtools-mcp` のツールリファレンスを参照してください。デフォルトでは無効になっており、明示的に有効化が必要です。

**WebMCP デバッグ**

WebMCP（Web Model Context Protocol）は、ウェブページが LLM エージェント向けにカスタムツールを登録できる新しい標準です。現在オリジントライアル中で、`chrome://flags` から以下のフラグを有効にすることでテストできます：

- `#devtools-webmcp-support`
- `#enable-webmcp-testing`

DevTools の「アプリケーション」パネルに新しいデバッグツールが追加されており、ページが公開しているツールの一覧確認、パラメータを指定した手動実行、ツール呼び出しのリアルタイム追跡が可能になります。

参考：[What's New in DevTools 149](https://developer.chrome.com/blog/new-in-devtools-149)

## 公式リソース

- ドキュメント：https://developer.chrome.com/docs/devtools/agents/get-started
- GitHub リポジトリ：https://github.com/ChromeDevTools/chrome-devtools-mcp
- v1 リリース発表：https://developer.chrome.com/blog/devtools-for-agents-v1
- Google I/O セッション動画：Supercharge your AI coding workflow with Chrome DevTools for agents

## まとめ

Chrome DevTools for Agents v1 を Claude Code に接続することで変わること：

- Lighthouse 監査が自動化され、品質ゲートとして組み込めます
- モバイル・低速回線のエミュレーションをプロンプト一行で実行できます
- 認証済みセッションをそのままエージェントに渡してデバッグできます
- メモリリーク検出をエージェントに委任できます
- プロトコル モニターでエージェントと Chrome の CDP 通信を可視化でき、トラブルシュートに使えます

「コードを書く AI」が「ブラウザの挙動を確認しながら修正する AI」になります。コードを書いて、すぐ動作を確認して、問題があれば直す——このサイクルをエージェントが一人で回せるようになります。

セットアップは 2 コマンドで完了するので、まず試してみてください。
