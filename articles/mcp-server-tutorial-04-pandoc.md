---
title: "【MCPのトリセツ #4】mcp-pandoc： AIでドキュメント形式を変換"
emoji: "🐸"
type: "tech"
topics: ["mcp", "pandoc", "github", "claude", "windsurf", "ai", "生成ai", "ai駆動開発"]
published: true
---

## 💡 MCPの始め方シリーズについて

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。今回は、強力なドキュメント変換ツールをAIから操作できるようになる mcp-pandoc の導入方法と活用法を解説します！

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. 👉 [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
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

## ✨ AIチャットでドキュメント変換： mcp-pandoc

AIとのチャットでいい感じの文章が生成された時、「これを Word 文書として保存したい」「HTMLに変換してウェブサイトに載せたい」と思ったことはありませんか？

mcp-pandoc はそれをできるようにしてくれる MCPサーバーです。Pandocという強力なドキュメント変換ツールをAIから操作できるようになり、テキストをさまざまな形式（txt、docx、epub、html、markdownなど）に変換できます。

## 🚀 mcp-pandoc でできること

このMCPサーバーを導入すると、次のような変換が可能になります：

- Markdown を Wordドキュメント（docx）に変換
- Markdown を ePub に変換して電子書籍化
- Markdown をPDF変換（※現在開発中の機能・2025年3月時点）
- Word文書を Markdown に変換
- プレーンテキストを HTMLに変換

## 👨‍💻 使用例（プロンプト）

mcp-pandoc は直感的に使えます。以下のようなプロンプトで文書変換ができます。
会話の中で良い文章ができたときに、そのまま指定のファイル形式で保存できるのが大きな魅力です。

### 1. マークダウンからWord文書を作成しデスクトップに保存

```text
これを docx に変換して、デスクトップフォルダにファイル名 "SampleDoc" で保存して。
```

### 2. マークダウンからHTMLを作成しデスクトップに保存

```text
html 形式に変換し、見やすいレイアウトとデザインにして、デスクトップフォルダに sample.html として保存して
```

```text
この Markdown形式の技術解説を、シンタックスハイライトとレスポンシブデザインを適用したHTMLに変換して、~/Projects/docs/technical-guide.html として保存して
```

### 3. マークダウンからPDFを作成し、元ファイルと同じ場所に保存

```text
PDF に変換して、元ファイルと同じ場所に保存して
```

### 4. マークダウンからビジネス文書を作成

```text
この会議メモをプロフェッショナルなWord文書に変換して、目次と見出しを適切に設定し、~/Documents/Business/meeting-report.docx として保存してください。
```

### 5. 電子書籍の作成

```text
この小説をePub形式に変換し、表紙画像を追加して、章ごとに適切に区切ってください。~/Documents/Books/my-novel.epub に保存してください。
```

### 6. 既存文書ファイルの変換

```text
/Users/yourname/Documents/report.md ファイルを読み込んで、Word形式に変換し、同じフォルダに report.docx として保存してください。
```

### 7. YouTube MCPサーバーとの組み合わせ

```text
1. このYouTube講義の内容を要約してください：https://www.youtube.com/watch?v=xxxxx
2. 要約をマークダウン形式でまとめてください
3. それをHTML形式に変換して、目次とセクション分けを追加し、~/Documents/lectures/summary.html として保存してください
```

## ⚠️ 使用上の注意点

mcp-pandoc を使用する際の重要なポイント：

1. **ファイルパスの指定**: ファイル名と拡張子を含む完全なファイルパスを指定する必要があります。
2. **PDF対応状況**: PDFサポートは現在開発中の機能です（2025年3月時点）。
3. **ディレクトリアクセス**: 保存先のディレクトリが [Filesystem MCP](./MCPサーバー%2001%20server-filesystem%20MCPサーバーでローカルファイルを読み書きできるようにする.md) でアクセス許可されていることを確認してください。

---

## 🛠️ インストール手順

※事前にローカルファイルシステムにアクセスするための [Filesystem MCP](./MCPサーバー%2001%20server-filesystem%20MCPサーバーでローカルファイルを読み書きできるようにする.md) をインストールしておいてください。

mcp-pandoc のセットアップは2ステップで完了します。

### 1. TeX Live のインストール（PDF変換に必要）

PDFへの変換機能を使用する場合、TeX Live という文書作成システムが必要です。
（※PDFサポートは現在開発中ですが、先にインストールしておくと便利です）

**macOSの場合：**

```bash
brew install texlive
```

**Windowsの場合：**
[TeX Liveの公式サイト](https://tug.org/texlive/)からインストーラーをダウンロードしてインストールします。

### 2. mcp-pandoc のインストール

#### 方法1： mcp-installer を使う場合（推奨）

前回または[シリーズ記事#0](./MCPサーバー%2000%20簡単に導入する手順%20\(mcp-installer\).md)で mcp-installer をセットアップ済みであれば、Claude に以下のように指示するだけでインストールできます。

```text
MCPサーバー mcp-pandoc をインストールして
```

#### 方法2：設定ファイルを直接編集する場合

Claude Desktop の設定ファイル（`~/Library/Application Support/Claude/claude_desktop_config.json`）を開き、以下のようにmcp-pandocの設定を追加します。(uv がインストールされている必要があります)。既に他のMCPサーバーを設定している場合は、"mcpServers" オブジェクト内に追加してください。

```json
{
  "mcpServers": {
    "mcp-pandoc": {
      "command": "uvx",
      "args": [
        "mcp-pandoc"
      ],
      "env": {}
    }
  }
}
```

設定後、Claude Desktop を再起動してください。

---

## 📝 まとめ

mcp-pandoc は、AI との会話で生まれたコンテンツを様々な形式で保存し、活用するための強力なツールです。Markdown、Word、HTML、ePubなど多くの形式に対応しており、ビジネス文書、技術ドキュメント、創作活動など、幅広い用途に使えます。

Filesystem MCP、YouTube MCPなど他のサーバーと組み合わせることで、さらに可能性が広がります。ぜひ mcp-pandoc を導入して、AIとの作業をもっと便利にしてみてください！

新しいMCP記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。フォローとツッコミ、お待ちしています！

## 📚 参考リンク

- [mcp-pandoc GitHub リポジトリ](https://github.com/vivekVells/mcp-pandoc)
- [Pandoc 公式サイト](https://pandoc.org/)
- [TeX Live 公式サイト](https://tug.org/texlive/)

次回の記事では、GithubリポジトリをAIとの会話で操作できる「[GitHub MCPサーバー](./mcp-server-tutorial-05-github)について解説します。
