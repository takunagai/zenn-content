---
title: "【MCPのトリセツ #10】Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う"
emoji: "🐸"
type: "tech"
topics: ["mcp", "raindrop", "claude", "claudecode", "codex"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズです。今回はブックマークサービス [Raindrop.io](https://raindrop.io/) の MCP サーバーを取り上げます。過去に保存したブックマークを、AI との会話から検索・整理できるようになります。

:::message
**更新日: 2026-09-22**（初版: 2025-03-08）

Raindrop.io が公式のリモート MCP サーバー（ベータ版・Pro プラン向け）を公開したため、公式版を中心に書き直しました。旧版で紹介したコミュニティ製サーバーは、無料プランで使う方法として残しています。
:::

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. [Slack MCPサーバー：チームコミュニケーションを強化](./mcp-server-tutorial-07-slack)
8. [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
9. [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. **Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う（この記事）**
11. [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
12. [Blender MCP Server: 会話で Blender を操作し3Dモデルを作成](./mcp-server-tutorial-12-blender)
13. [Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行](./mcp-server-tutorial-13-perplexity)
14. [国土交通省がMCPサーバーを公開：AI時代のオープンデータ活用45選](./mcp-server-tutorial-14-milt-data)

資料: [ウェブ情報を取得するMCPの比較 (Fetch、Firecrawl、Markdownify、Perplexity)](./mcp-server-tutorial-reference-web-mcp)

---

## 2 つの選択肢

| 観点 | 公式リモートサーバー | コミュニティ製サーバー |
|---|---|---|
| 提供元 | Raindrop.io | [hiromitsusasaki/raindrop-io-mcp-server](https://github.com/hiromitsusasaki/raindrop-io-mcp-server) |
| 対象プラン | Pro（ベータ版） | 無料プランでも可 |
| 導入 | URL を登録して OAuth でログイン | クローンしてビルド、テストトークンを設定 |
| できること | ブックマーク・コレクション・タグ・ハイライトの検索、作成、更新、削除 | ブックマークの検索と作成、コレクションの一覧 |

Pro プランを使っているなら、公式版が第一候補です。ビルドもトークンの管理も要りません。

## 公式リモートサーバーを使う

接続先は `https://api.raindrop.io/rest/v2/ai/mcp` です。Streamable HTTP で接続し、初回にブラウザで Raindrop.io へのアクセスを許可します（OAuth 2.1）。

### Claude Desktop

サイドバーの「Customize」から「Connectors」を開き、「+」を押します。一覧に Raindrop.io がない場合は「Add custom connector」を選び、上の URL を登録します。

### Claude Code

```bash
claude mcp add --transport http raindrop https://api.raindrop.io/rest/v2/ai/mcp
```

登録後、Claude Code 内で `/mcp` を実行し、`raindrop` を選んで認証します。

### Codex

```bash
codex mcp add raindrop --url https://api.raindrop.io/rest/v2/ai/mcp
codex mcp login raindrop
```

リモート接続に対応していないクライアントでは、`npx -y mcp-remote https://api.raindrop.io/rest/v2/ai/mcp` をローカルサーバーのコマンドとして登録する方法が公式ヘルプで案内されています。

### 公式版でできること

| 分類 | 操作 |
|---|---|
| ブックマーク | 検索、本文の取得、作成、更新、削除。分類やタグが合っていないものの検出 |
| コレクション | 検索、作成、更新、統合、削除 |
| タグ | 検索、名前の変更と統合、削除 |
| ハイライト | 検索、作成、更新、削除 |

削除や統合もできるので、整理を任せるときは、先に「何をどう変えるか」の案を出させてから実行させるのが安全です。

## コミュニティ製サーバーを使う（無料プラン向け）

### アクセストークンの発行

1. [Raindrop.io の設定画面](https://app.raindrop.io/settings/integrations)を開く
2. 「開発者向け」の欄で新しいアプリを作成する（名前は `mcp-server` など）
3. 作成したアプリを開き、「Create test token」を押してトークンを控える

### インストール

```bash
cd ~/tools/mcp-server
git clone https://github.com/hiromitsusasaki/raindrop-io-mcp-server
cd raindrop-io-mcp-server
npm install
npm run build
```

### Claude Desktop

設定ファイル（開き方は[シリーズ #1](./mcp-server-tutorial-01-install) を参照）に次を追加します。`args` にはビルドでできた `build/index.js` の絶対パスを書きます。

```json
{
  "mcpServers": {
    "raindrop": {
      "command": "node",
      "args": ["/Users/yourname/tools/mcp-server/raindrop-io-mcp-server/build/index.js"],
      "env": {
        "RAINDROP_TOKEN": "your_access_token_here"
      }
    }
  }
}
```

トークンは設定ファイルに平文で残ります。チャット欄には貼らず、不要になったら設定画面でアプリごと削除してください。

提供されるツールは `search-bookmarks`（検索）、`list-collections`（コレクション一覧）、`create-bookmark`（作成）の 3 つです。

## プロンプトのサンプル

### 検索する

```text
Raindrop で「MCP」に関するブックマークを探して、保存日の新しい順に 10 件見せて
```

```text
タグ「design」が付いていて、github.com のドメインのブックマークを探して
```

```text
コレクションの一覧を見せて。そのあと「開発」コレクションの中を「tutorial」で検索して
```

### 保存した情報を使う

```text
過去に保存した「料金設計」関連の記事を探して、それぞれの要点を 2 行でまとめて。記事を書くときの参考資料の一覧にしたい
```

```text
先月保存したブックマークを分野別に分けて、どんなテーマに関心が偏っているか教えて
```

### 追加する

```text
https://developer.mozilla.org を Raindrop に追加して。タイトルは「MDN Web Docs」、タグは web と documentation
```

### 整理する（公式版）

```text
タグの一覧を見て、表記ゆれ（「javascript」と「JavaScript」など）を洗い出して。統合の案を出すだけで、実行はまだしないで
```

```text
コレクション「未整理」のブックマークを、内容に合うコレクションへ振り分ける案を出して。私が確認してから移動して
```

## 使用上の注意点

- **整理は案を見てから**: 公式版は更新・削除・統合ができます。まとめて実行させる前に、対象と変更内容を一覧で確認します
- **本文の取得は外部の入力**: ブックマーク先のページ本文を AI に読ませると、ページに紛れた AI 向けの指示を拾う可能性があります。読んだ直後に想定外の操作をしようとしたら、承認せずに止めます
- **ベータ版である点**: 公式版はベータ版です。ツールの構成や対象プランは変わる可能性があるので、導入時に公式ヘルプを確認してください

## まとめ

- Raindrop.io 公式のリモート MCP サーバーが登場しました。Pro プランなら URL を登録して OAuth でログインするだけで使えます
- 無料プランでは、コミュニティ製サーバーで検索と追加ができます
- 公式版は削除や統合もできます。整理を任せるときは、案を確認してから実行させます

新しい MCP 記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。

## 参考リンク

- [MCP server - Raindrop.io Help](https://help.raindrop.io/integrations/mcp)
- [MCP - Raindrop.io Developer Docs](https://developer.raindrop.io/mcp/mcp)
- [hiromitsusasaki/raindrop-io-mcp-server - GitHub](https://github.com/hiromitsusasaki/raindrop-io-mcp-server)
- [Raindrop.io 公式サイト](https://raindrop.io/)

次回は、Web ページの内容を取得する「[Fetch MCP Server](./mcp-server-tutorial-11-fetch)」を解説します。
