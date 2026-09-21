---
title: "【MCPのトリセツ #7】Slack MCPサーバー：チームコミュニケーションを強化"
emoji: "🐸"
type: "tech"
topics: ["mcp", "slack", "claude", "claudecode", "ai"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズです。今回は Slack 公式の MCP サーバーを取り上げます。過去のやり取りの検索と要約、メッセージの下書きと投稿を、AI との会話から行えるようになります。

:::message
**更新日: 2026-09-22**（初版: 2025-03-08）

旧版で紹介していた `@modelcontextprotocol/server-slack` は公式リポジトリからアーカイブされたため、Slack 公式のリモート MCP サーバーの手順に差し替えています。Bot アプリの作成やチーム ID の取得は不要になりました。
:::

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. **Slack MCPサーバー：チームコミュニケーションを強化（この記事）**
8. [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
9. [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)
11. [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
12. [Blender MCP Server: 会話で Blender を操作し3Dモデルを作成](./mcp-server-tutorial-12-blender)
13. [Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行](./mcp-server-tutorial-13-perplexity)
14. [国土交通省がMCPサーバーを公開：AI時代のオープンデータ活用45選](./mcp-server-tutorial-14-milt-data)

資料: [ウェブ情報を取得するMCPの比較 (Fetch、Firecrawl、Markdownify、Perplexity)](./mcp-server-tutorial-reference-web-mcp)

---

## Slack MCP でできること

Slack の情報は流れていくので、「あの件、どこで決まったんだっけ」を探す時間がかかります。Slack MCP サーバーをつなぐと、AI が自分の権限の範囲で Slack を検索し、文脈を踏まえて答えてくれます。

| 分類 | できること |
|---|---|
| 検索 | メッセージとファイルを日付・ユーザー・種類で絞り込む。ユーザー、チャンネル、カスタム絵文字の検索 |
| メッセージ | チャンネル履歴とスレッドの読み取り、送信、下書き、リアクション |
| canvas | 整形されたドキュメントの作成と共有、Markdown での書き出し |
| ユーザー | プロフィールの取得、チャンネルメンバーの一覧 |
| ファイル・リスト | ファイルのアップロード、リストの作成と読み取り |

## 公式リモートサーバーの仕組み

Slack 公式の MCP サーバーは `https://mcp.slack.com/mcp` で提供されるリモートサーバーです。旧来の Bot Token 方式と違い、OAuth で自分の Slack アカウントとしてログインします。AI が読めるのは、自分が Slack 上で見られる範囲だけです。

導入前に押さえておく点が 2 つあります。

- **ワークスペース管理者の承認が要る**: MCP クライアント（Claude など）は Slack アプリとして扱われ、通常のアプリ承認の手続きに乗ります。個人の判断だけでは接続できないワークスペースもあります
- **接続できるクライアントが決まっている**: Slack のドキュメントに載っている対応クライアントは、Claude.ai、Claude Code、Perplexity、Cursor です（2026 年 9 月時点）

## セットアップ手順

### Claude Desktop

1. サイドバーの「Customize」を開く
2. 「Connectors」を開き、「+」を押す
3. Slack を選び、OAuth 認証を済ませる

認証画面で接続先のワークスペースを選びます。管理者の承認が済んでいない場合は、ここで承認リクエストを送ることになります。

### Claude Code

公式プラグインを入れます。リモート MCP サーバーの URL や OAuth の設定が自動で入ります。

```bash
claude plugin install slack
```

Claude Code の中から入れる場合は `/plugin install slack` です。初回にツールを使うとき、ブラウザで Slack の認証画面が開きます。

### Codex

Codex は Slack 公式の対応クライアントに含まれていません。Codex から Slack を扱いたい場合は、コミュニティ製のローカルサーバーを使うことになります。旧リファレンス実装を引き継いだ [zencoderai/slack-mcp-server](https://github.com/zencoderai/slack-mcp-server) や、[korotovsky/slack-mcp-server](https://github.com/korotovsky/slack-mcp-server) があります。

これらは Bot Token やユーザートークンを自分で発行して設定する方式です。トークンの管理と、ワークスペースの規約に沿っているかの確認は利用者の責任になります。業務のワークスペースで使うなら、先に管理者へ相談するのが無難です。

## 基本的な使い方（プロンプト）

```text
#general チャンネルの直近 1 週間の会話を要約して
```

```text
#project-alpha で「リリース計画」について議論しているスレッドを探して、決定事項をまとめて
```

```text
先月、田中さんが共有してくれた見積もりのファイルを探して
```

```text
#announcements に投稿する、明日の全体ミーティングの案内文を下書きして。送信はせず、下書きとして保存して
```

投稿までさせる場合も、まず下書きで止めて、文面を確認してから送るのが安全です。AI の文章がそのままチームに流れるのを防げます。

## 実践的な活用例

### 会議の決定事項を別チャンネルへ共有する

```text
昨日の #team-meetings での製品企画会議の内容を要約し、決定事項と次のアクションをリストにして、#product-dev に共有する文面を下書きして
```

### 過去の議論から判断の経緯を掘り起こす

```text
過去 6 か月の #engineering から、システムアーキテクチャに関する重要な議論と決定事項を時系列で抽出して
```

### 新メンバー向けの資料を canvas にまとめる

```text
#general と #help でよく出る質問と回答を整理して、新メンバー向けの FAQ を canvas で作成して
```

### チーム間の情報共有を助ける

```text
#marketing で議論されている新機能のプロモーション計画を要約して。開発チーム向けに、対応が必要な点を先頭に書いて
```

## 他の MCP サーバーとの組み合わせ

[Filesystem MCP](./mcp-server-tutorial-02-filesystem) と組み合わせると、Slack の議論を手元の文書に残せます。

```text
#project-docs で共有された最新の仕様の議論を整理し、要点を ~/Documents/Projects/summary.md に保存して
```

[GitHub MCP](./mcp-server-tutorial-05-github) と組み合わせると、Slack で報告されたバグを Issue にできます。

```text
#dev-team で今週報告されたバグを一覧にして、まだ Issue になっていないものを myusername/our-app の Issue の下書きにして
```

## 安全に使うための注意点

- **読める範囲を意識する**: AI は自分のアカウントで見える DM やプライベートチャンネルも検索できます。機密度の高い会話を AI に渡してよいか、社内のルールを先に確認してください
- **チームに周知する**: AI が過去の会話を読んで要約する使い方をしていることを、メンバーと共有しておくと行き違いが起きません
- **送信は確認してから**: 送信・リアクションなどの書き込み系ツールは自動承認にせず、内容を見てから許可します
- **外部から来た文章に注意する**: 共有チャンネルや外部連携で流れてくるメッセージには、AI への指示を装った文章が紛れる可能性があります。検索結果をもとに AI が想定外の操作をしようとしたら、承認せずに止めます

## まとめ

- 旧 `@modelcontextprotocol/server-slack` はアーカイブされました。現在は Slack 公式のリモート MCP サーバー（`https://mcp.slack.com/mcp`）を使います
- Bot アプリの作成は不要で、OAuth で自分のアカウントとして接続します。ワークスペース管理者の承認が前提です
- Claude Desktop はコネクタ、Claude Code は公式プラグインで導入します。Codex は公式の対応クライアントに含まれません

新しい MCP 記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。

## 参考リンク

- [Slack MCP server - Slack Developer Docs](https://docs.slack.dev/ai/slack-mcp-server/)
- [Connect to Claude - Slack Developer Docs](https://docs.slack.dev/ai/slack-mcp-server/connect-to-claude/)
- [modelcontextprotocol/servers-archived - GitHub](https://github.com/modelcontextprotocol/servers-archived)
- [zencoderai/slack-mcp-server - GitHub](https://github.com/zencoderai/slack-mcp-server)
- [korotovsky/slack-mcp-server - GitHub](https://github.com/korotovsky/slack-mcp-server)

次回は、ウェブ情報をスクレイピングして分析できる「[Firecrawl MCP](./mcp-server-tutorial-08-firecrawl)」を解説します。
