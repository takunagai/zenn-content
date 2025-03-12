---
title: "【MCPのトリセツ #9】Markdownify MCP Server: WebページやPDFをMarkdown文書化"
emoji: "🐸"
type: "tech"
topics: ["mcp", "markdownify", "markdown", "claude", "windsurf", "ai", "生成ai", "ai駆動開発"]
published: true
---

## 💡 MCPの始め方シリーズについて

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。今回は、Markdownify MCPサーバーの導入方法と活用テクニックを紹介します。ウェブページやPDFをAIとの会話でMarkdown文書化することで、リサーチや情報収集を効率化できます！

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. [Slack MCPサーバー：チームコミュニケーションを強化](./mcp-server-tutorial-07-slack)
8. [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
9. 👉 [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)
11. [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
12. [Blender MCP Server: 会話で Blender を操作し3Dモデルを作成](./mcp-server-tutorial-12-blender)
13. [ウェブの情報を取得するMCPの使い分け (Fetch、Firecrawl、Markdownify)](./mcp-server-tutorial-reference-web-mcp)

---

[zcaceres/markdownify-mcp - Github](https://github.com/zcaceres/markdownify-mcp)

## 🚀 Markdownify MCP Server でできること

- 複数のファイルタイプを Markdown に変換
  - PDF、画像、オーディオ（転写付き）、docx、XLSX、PPTX
- WebコンテンツをMarkdownに変換
  - YouTube 字幕
  - Bing 検索結果
  - 一般的な Webページ
- 既存の Markdownファイルを取得

> 📝 **ノート**: Fetch MCP、Firecrawl MCP、Markdownify MCPの比較と使い分けについては、[ウェブの情報を取得するMCPの使い分け](./mcp-server-tutorial-reference-web-mcp)を参照してください。

## 👨‍💻 Markdownify MCP Server プロンプトのサンプル

### 1. 基本的な URL から Markdown への変換

```
以下URLの内容をMarkdownに変換して：
https://example.com/blog/article-123
```

### 2. 複数のWebページを一括変換

```
以下の複数のURLをMarkdownに変換し、それぞれのコンテンツを見出しで区切って：
- https://site1.com/article1
- https://site2.com/article2
- https://site3.com/article3
```

### 3. 特定の要素に焦点を当てた変換

```
次の技術ブログから、コードスニペットと主要な見出しだけを抽出し、Markdownに変換して：
https://tech-blog-example.com/tutorial/javascript-basics
```

### 4. フォーマット指定付き変換

```
以下のニュースサイトの記事を以下の条件でMarkdownに変換して：
- 見出しは ## レベルで統一
- 引用部分は > でマーク
- リンクはそのまま保持
- 画像は ![キャプション](URL) 形式に統一

URL: https://news-example.com/technology/latest-trends
```

### 5. コンテンツフィルタリング付き変換

```
以下URLからAIに関連する内容だけを抽出してMarkdownに変換して。
関連性の低いセクションは除外して
https://news.ycombinator.com/best
```

### 6. PDF から Markdown への変換

```
添付のPDFをMarkdownに変換して。
目次構造を保持し、表はMarkdown形式のテーブルとして変換して
```

### 7. YouTube 動画の字幕抽出

```
以下のYouTube動画の字幕をMarkdown形式で抽出して。
タイムスタンプを含め、主要なポイントには見出しを付けて：
https://www.youtube.com/watch?v=example12345
```

### 8. 技術文書の整形済み変換

```
次の技術ドキュメントをMarkdownに変換し、以下の形式に整えて：
- コードブロックは言語指定付きで
- API エンドポイントは表形式で整理
- パラメータ説明はリスト形式で
- 重要な警告は強調表示

URL: https://api-docs-example.com/reference
```

### 9. コンテンツ要約付き変換

```
以下の長文記事をMarkdownに変換し、各セクションの冒頭に2-3文の要約を追加して：
https://longform-content.com/comprehensive-guide
```

### 10. 多言語コンテンツの処理

```
以下の英語のウェブサイトをMarkdownに変換し、日本語への翻訳も併記して：
https://english-site.com/global-news
```

### 11. アカデミック論文の構造化変換

```
以下の学術論文をMarkdownに変換し、次の構造を明確にして：
- 要旨（Abstract）を引用形式で
- 方法論、結果、考察を別々のセクションで
- 参考文献を番号付きリストで
- 図表に連番を振って参照しやすく

URL: https://academic-journal.edu/paper/12345
```

### 12. SEO 分析付き変換

```
次のランディングページをMarkdownに変換し、SEO観点からの分析を追加して：
- H1, H2 の見出し構造
- メタタグ情報
- キーワード密度
- 内部リンク・外部リンクの数と質

URL: https://business-site.com/services
```

### 13. ニュースレター作成用フォーマット

```
以下の複数のニュース記事を取得し、週刊ニュースレター形式のMarkdownに変換して：
- トップニュース（3件）を冒頭に要約
- カテゴリ別に記事をグループ化
- 各記事に 「続きを読む」 リンクを追加
- 最後に今週の重要なイベントカレンダーを表形式で

URLs:
- https://news1.com/article1
- https://news2.com/article2
- https://news3.com/article5
```

### 14. プログラミングチュートリアルの強化

```
次のプログラミングチュートリアルをMarkdownに変換し、以下を追加して：
- コードブロックに構文ハイライトを適用
- 重要な概念に注釈を追加
- 前提知識と「次に学ぶべきこと」のセクションを追加
- コードの各部分の説明を追加

URL: https://dev-tutorial.com/advanced-patterns
```

### 15. 製品比較表の作成

```
以下の複数の製品レビューページから情報を抽出し、Markdown形式の比較表を作成して：
- 製品名、価格、主な機能、長所、短所の列を含む
- 価格帯で製品を並べ替え
- 各製品の総合評価（5段階）を追加

URLs:
- https://review-site.com/product1
- https://review-site.com/product2
- https://review-site.com/product3
```

### 16. ウェビナー内容の構造化記録

```
次のウェビナー録画の内容をMarkdown形式で文書化して：
- 主要なポイントを見出しとして
- Q&A セクションを別に整理
- デモンストレーションの手順をステップバイステップのリストとして
- 参照されたリソースのリンク集を最後に追加

URL: https://webinar-platform.com/recording/12345
```

---

## 🛠️ インストールと設定

※ 試していませんが、npm でもいけるはず。

インストールするディレクトリに移動
`cd ~/tools/mcp-server`
(場所はお好みで。自分は上記のパスを MCPサーバー置き場としている)

AI作業用フォルダにクローン
`git clone git@github.com:zcaceres/markdownify-mcp.git`

中に移動
`cd markdownify-mcp`

インストール
`pnpm install`

ビルド
`pnpm run build`

MCPサーバーを起動 (Claude Desktop の再起動でOK)
`pnpm start`

### 設定

“args" にインストールしたフォルダ内にある index.js の絶対パスを設定。
“env” の uv のパスは、デフォルト設定なら不要 (必要な場合は `which uv` で場所を確認できる)。

```claude_desktop_config.json
{
  "mcpServers": {
    "markdownify": {
      "command": "node",
      "args": [
        "{ABSOLUTE PATH TO FILE HERE}/dist/index.js"
      ],
      "env": {
        // By default, the server will use the default install location of `uv`
        "UV_PATH": "/path/to/uv"
      }
    }
  }
}
```

---

## 📝 まとめ

Markdownify MCPサーバーは、ウェブページやPDFなど様々な形式のコンテンツをMarkdown形式に変換できる強力なツールです。この機能により、以下のようなメリットが得られます：

- 複数の情報源（ウェブページ、PDF、動画字幕など）を統一されたMarkdown形式で管理
- 情報の整理・構造化が容易になり、後からの参照や編集が効率化
- AIとの対話を通じて、コンテンツの変換だけでなく要約や分析も同時に実行可能
- 様々なフォーマット指定やフィルタリングオプションで、必要な情報だけを抽出可能

特に研究、コンテンツ制作、技術文書作成、情報整理などの作業において、Markdownify MCPは大きな時間節約と効率化をもたらします。他のMCPサーバーと組み合わせることで、情報収集から整理、分析、保存までの一連のワークフローを自動化することも可能です。

ぜひMarkdownify MCPを導入して、情報処理ワークフローを効率化してみてください！

## 📚 参考リンク

- [markdownify-mcp GitHub リポジトリ](https://github.com/zcaceres/markdownify-mcp)

次回の記事では、[Raindrop.io MCPサーバー](./mcp-server-tutorial-10-raindropio)を紹介します。お楽しみに！
