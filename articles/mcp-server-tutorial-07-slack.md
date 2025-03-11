---
title: "【MCPのトリセツ #7】Slack MCPサーバー：チームコミュニケーションを強化"
emoji: "🐸"
type: "tech"
topics: ["mcp", "slack", "claude", "windsurf", "ai", "生成ai", "ai駆動開発"]
published: true
---

## 💡 MCPの始め方シリーズについて

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。今回は、Slack MCPサーバーの導入方法と活用法を解説します。

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. 👉 [Slack MCPサーバー：チームコミュニケーションを強化](./mcp-server-tutorial-07-slack)
8. [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
9. [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)
11. [ウェブの情報を取得するMCPの使い分け (Fetch、Firecrawl、Markdownify)](./mcp-server-tutorial-reference-web-mcp)

---

## 🚀 Slack MCPでできること

「チームの Slack で過去のやり取りを分析して課題を見つけたい...」
「Slack のメッセージやスレッドをAIに参照してもらい、的確な返答を得たい...」
「AI がチームの会話コンテキストを理解した上でアドバイスしてくれたら...」

Slack MCPサーバーを使えば、これらを実現できます。AI と Slack を連携させ、チームコミュニケーションの分析や効率化を目指しましょう。

- チャンネル内の会話履歴を参照
- 特定のトピックに関する過去の議論を要約
- メッセージの送信や返信
- チャンネルの作成や管理
- メンバーへのメンション
- チーム内の知識やノウハウの抽出
- コミュニケーションの傾向分析
- 効果的な会議要約の作成と共有

---

## 🛠️ セットアップ手順

Slack MCPサーバーをセットアップするには、以下の手順に従います。

### 1. Slack Bot アプリの作成とトークンの取得

まず、Slackワークスペースで使用するBotを作成し、必要なトークンを取得します：

1. [Slack API](https://api.slack.com/apps)のページにアクセスしてログイン
2. 「Create an App」をクリック
3. 「From scratch」を選択し、以下を入力：
   - **アプリ名**: 任意の名前（例：「Claude MCP Bot」）
   - **ワークスペース**: 対象のSlackワークスペース
4. 「Create App」をクリック
5. 左サイドバーから「OAuth & Permissions」を選択
6. 「Scopes」セクションで「Bot Token Scopes」に以下の権限を追加：
   - `chat:write` (メッセージ送信)
   - `channels:read` (チャンネル情報読み取り)
   - `channels:history` (チャンネル履歴閲覧)
   - `users:read` (ユーザー情報読み取り)
   - `team:read` (チーム情報読み取り)
   - 必要に応じて他の権限も追加
7. 画面上部の「Install to Workspace」ボタンをクリック
8. 認証画面で「許可する」をクリック
9. 「Bot User OAuth Token」（`xoxb-`で始まるトークン）をコピー

### 2. チームIDの取得

SlackチームのチームID（Team ID）も必要です：

1. ブラウザでSlackにログイン
2. URLを確認：`https://app.slack.com/client/T0XXXXXXX/...`
3. URLの「T0XXXXXXX」部分がチームIDです

### 3. MCPサーバーの設定

取得したトークンとチームIDを使って、Slack MCPサーバーを設定します。

#### Slack MCPサーバーの設定

##### 方法1： mcp-installerを使う場合

[シリーズ記事#0](./MCPサーバー%2000%20簡単に導入する手順%20\(mcp-installer\).md)でmcp-installerをセットアップ済みであれば、Claude に以下のように指示できます。※セキュリティ的に、チャットでのAPIキーなどの直接入力は絶対に避けてください。

```
MCPサーバー @modelcontextprotocol/server-slack をインストールして。環境変数 SLACK_BOT_TOKEN は 'your-bot-token-here' で、SLACK_TEAM_ID は 'T0XXXXXXX' で設定します
```

その後、Claude Desktopの設定ファイル（`~/Library/Application Support/Claude/claude_desktop_config.json`）を開き、`your-bot-token-here` と `T0XXXXXXX` の部分を、実際に取得したトークンとチームIDに置き換えてください。

##### 方法2：設定ファイルを直接編集する場合

Claude Desktopの設定ファイル（`~/Library/Application Support/Claude/claude_desktop_config.json`）を開き、以下のように設定を追加します。`your-bot-token-here` と `T0XXXXXXX` の部分は、実際に取得したトークンとチームIDに置き換えてください。

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-slack"
      ],
      "env": {
        "SLACK_BOT_TOKEN": "your-bot-token-here",
        "SLACK_TEAM_ID": "T0XXXXXXX"
      }
    }
  }
}
```

#### Windsurf の場合

`~/.codeium/windsurf/mcp_config.json` を開き、上と同様の設定を追加します。

---

## 👨‍💻 基本的な使い方 (プロンプト)

### 指定したチャンネルの最新メッセージを読み取り、要約

```
#general チャンネルの最近の会話を要約して
```

### 該当するキーワードを含むメッセージスレッドを見つけ、内容を要約

```
#project-alpha チャンネルで「リリース計画」に関する議論を見つけて要約して
```

### メッセージを作成して、指定したチャンネルに投稿

```
#announcements チャンネルに、明日の全体ミーティングの案内を丁寧な口調で投稿して
```

### 新しいチャンネルを作成し、初期メッセージを投稿

```
新しいプロジェクト「Beta」用のチャンネル #project-beta を作成し、初回メッセージとしてプロジェクト概要を投稿してください
```

## 💡 実践的な活用例 (プロンプト)

### 1. 会議の内容を要約して、重要ポイントをまとめて別チャンネルに共有

```
昨日の #team-meetings チャンネルでの製品企画会議の内容を要約し、決定事項と次のアクションアイテムをリスト化して #product-dev チャンネルに共有して
```

### 2. チャンネル履歴から特定トピックに関する重要情報を抽出

```
過去6ヶ月間の #engineering チャンネルから、システムアーキテクチャに関する重要な議論と決定事項を抽出して
```

### 3. 新入社員のオンボーディング支援

```
新入社員向けに、#general チャンネルと #random チャンネルの雰囲気やよく話題になるトピック、チームカルチャーについて分析して
```

### 4. コミュニケーションパターンを分析し、改善点を提案

```
#project-gamma チャンネルでの議論の傾向を分析し、意思決定プロセスを効率化するための提案をして
```

### 5. 異なるチーム間での情報共有の補助

```
#marketing チャンネルで議論されている新機能のプロモーション計画を要約し、#development チャンネルにわかりやすく共有して
```

## 🔄 他のMCPサーバーとの組み合わせ

Slack MCPサーバーは他のMCPサーバーと組み合わせることでさらに便利になります：

### チャンネルの内容を分析してファイルに保存 (Slack MCP + Filesystem MCP)

```
#project-docs チャンネルで共有された最新の仕様書の内容を分析し、その要点を ~/Documents/Projects/summary.md ファイルに保存してください
```

### チャンネルの議論内容を元に GitHubイシューを立てる (Slack MCP + GitHub MCP)

```
#dev-team チャンネルで議論されたバグ修正の内容をもとに、GitHubリポジトリ「our-app」にIssueを作成してください
```

### チャンネルで共有されたYouTube動画の要約 (Slack MCP + YouTube MCP)

```
#learning-resources チャンネルで共有されたYouTubeチュートリアルのリンクを見つけて、その内容を要約してください
```

## ⚠️ Slack MCPサーバーを安全に使用するための注意点

1. **最小権限の原則**: BotアプリにはChatの読み書きなど必要最小限の権限だけを与えましょう
2. **トークンの管理**: Slack Bot Token は機密情報として適切に管理し、公開リポジトリに入れないようにしましょう
3. **利用目的の明確化**: チームメンバーには AI が過去の会話を読めることを透明化し、利用目的や範囲を明確に共有しましょう
4. **プライバシー配慮**: DMやプライベートチャンネルへのアクセスは特に慎重に検討しましょう

## 📝 まとめ

Slack MCPサーバーを導入することで、AIはチームのコミュニケーションコンテキストを理解し、より的確なサポートが可能になります。会議の要約、知識の抽出、情報共有の効率化など、ビジネスコミュニケーションの多くの側面を改善できるでしょう。

特にリモートワークやグローバルチームが増える中で、Slack MCPサーバーはコミュニケーションギャップを埋め、チームの結束力を高める強力なツールとなります。プライバシーやセキュリティに配慮しながら、ぜひ Slack MCP を活用してみてください！

新しいMCP記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。フォローとツッコミ、お待ちしています！

## 📚 参考リンク

- [Slack API公式ドキュメント](https://api.slack.com/docs)
- [Slack App開発ガイド](https://api.slack.com/start/overview)
- [@modelcontextprotocol/server-slack](https://github.com/modelcontextprotocol/servers/tree/main/src/slack)
- [Windsurf MCP入門ガイド動画](https://www.youtube.com/watch?v=Y_kaQmhGmZk)

次回の記事では、ウェブ情報をスクレイピングして分析できる「[Firecrawl MCP](./mcp-server-tutorial-08-firecrawl)」について解説します。お楽しみに！
