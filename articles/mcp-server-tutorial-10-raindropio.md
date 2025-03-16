---
title: "【MCPのトリセツ #10】Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う"
emoji: "🐸"
type: "tech"
topics: ["mcp", "raindrop", "claude", "windsurf", "ai", "生成ai", "ai駆動開発"]
published: true
---

## 💡 MCPの始め方シリーズについて

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。今回は、Raindrop.io MCPサーバーの導入方法と活用テクニックを紹介します。AI との会話で Raindrop.io のブックマークにアクセスできるので、過去にマークしていた情報を容易に引き出せるようになります！

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
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)
11. [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
12. [Blender MCP Server: 会話で Blender を操作し3Dモデルを作成](./mcp-server-tutorial-12-blender)
13. [Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行](./mcp-server-tutorial-13-perplexity)

参考: [ウェブの情報を取得するMCPの使い分け (Fetch、Firecrawl、Markdownify)](./mcp-server-tutorial-reference-web-mcp)

---

[hiromitsusasaki/raindrop-io-mcp-server - Github](https://github.com/hiromitsusasaki/raindrop-io-mcp-server)

## 🚀 Raindrop.io MCP Server でできること

Raindrop.io のブックマークサービスと連携し、以下のことができます。

- ブックマークの検索（search-bookmarks）
- コレクションの一覧表示（list-collections）
- ブックマークの作成（create-bookmark）

## 👨‍💻 Raindrop.io MCP Server プロンプトのサンプル

### ブックマークを検索する

```text
「python」というキーワードでブックマークを検索して
```

```text
以下の条件でブックマークを検索して：
キーワード: python
並び順: 更新日時の新しい順
表示件数: 10件
```

### タグで検索する

```text
「programming」と「tutorial」のタグが付いたブックマークを探して
```

### 複数条件での詳細検索

```text
以下の条件でブックマークを検索して：
キーワード: machine learning
タグ: AI, tutorial
ページ: 1
表示数: 10
並び順: 作成日の新しい順
```

### 単語全体一致検索

```text
「python」という単語だけに完全一致するブックマークを検索して
（「python3」や「pythonista」は含まない）
```

### 指定ドメインで検索

```text
「github.com」ドメインのブックマークだけを表示して
```

### ドメインによる並べ替え

```text
「programming」タグのブックマークをドメイン名順で表示して
```

### 作成日や更新日での並べ替え

```text
すべてのブックマークを更新日の古い順で表示して
```

### コレクション一覧を表示する

```text
Raindropのコレクション一覧を表示して
```

### 特定のコレクション内を検索する

```text
コレクションID「12345」内で「デザイン」に関するブックマークを検索して
```

### コレクション情報と検索の組み合わせ

```text
まずコレクション一覧を表示して、その後で最も大きいコレクション内を「tutorial」で検索して
```

## ブックマークを追加

```text
https://example.com というURLをRaindropに「サンプルサイト」というタイトルで保存して
```

```text
次のURLをブックマークに追加して：
URL: https://developer.mozilla.org
タイトル: MDN Web Docs
タグ: web, programming, documentation
```

### 特定のコレクションにブックマークを追加

```text
https://www.wikipedia.org をコレクションID「12345」に「Wikipedia」というタイトルで保存して
```

## 🛠️ インストールと設定

インストールするディレクトリに移動

```bash
cd ~/tools/mcp-server
```

AI作業用フォルダにクローン

```bash
git clone https://github.com/hiromitsusasaki/raindrop-io-mcp-server
```

中に移動

```bash
cd raindrop-io-mcp-server
```

インストール

```bash
npm install
```

アクセストークンを発行し環境変数を設定する。

まずは、[Raindrop.io の設定画面](https://app.raindrop.io/settings/integrations)の 統合パネル の "開発者向け" で "作成 new app" ボタン。判別しやすい名前を付け(例：mcp-server) "作成" ボタン。作成されたものをクリック、"Create test token" リンクをクリックで、アクセストークンが生成されるのでそれをコピー。

そして、.env を作成し、上記で得たアクセストークンを環境変数に設定する。

```bash
touch .env
```

```bash
echo 'RAINDROP_TOKEN=your_access_token_here' >> .env
```

ビルド

```bash
npm run build
```

MCPサーバーを起動 (Claude Desktop 再起動で OK)

```bash
npm start
```

### Claude デスクトップ側の設定

"args" にインストールしたフォルダ内にある index.js の絶対パスを設定。

```json
{
  "mcpServers": {
    "raindrop": {
      "command": "node",
      "args": ["PATH_TO_BUILD/index.js"],
      "env": {
        "RAINDROP_TOKEN": "your_access_token_here"
      }
    }
  }
}
```

## 📝 まとめ

Raindrop.io MCP Serverは、AIとRaindrop.ioブックマークサービスを連携させる強力なツールです。この連携により、以下のようなメリットが得られます：

- AIとの対話だけで、過去に保存したブックマークに簡単にアクセス可能
- キーワード、タグ、ドメインなど多様な条件での柔軟な検索
- 会話の流れの中で新しいブックマークを追加できる利便性
- コレクション管理を通じた情報の整理と活用

特に情報収集や研究、プロジェクト管理などの場面で、Raindrop.io MCPは過去に保存した有用な情報を素早く引き出し、現在の作業に活かすことを可能にします。他のMCPサーバーと組み合わせることで、情報の収集、保存、検索、活用までの一連のワークフローをさらに効率化できます。

ぜひRaindrop.io MCPを導入して、ブックマーク管理をAIと連携させた新しい情報活用の形を体験してみてください！

## 📚 参考リンク

- [raindrop-io-mcp-server GitHub リポジトリ](https://github.com/hiromitsusasaki/raindrop-io-mcp-server)
- [Raindrop.io 公式サイト](https://raindrop.io/)
- [Raindrop.io API ドキュメント](https://developer.raindrop.io/)

これでMCPサーバー入門シリーズは一旦完結です。シリーズの最初から読み直したい方は[MCPの概要と導入方法](./mcp-server-tutorial-01-install)からどうぞ！
