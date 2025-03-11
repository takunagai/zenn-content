---
title: "【MCPのトリセツ リファレンス】ウェブの情報を取得するMCPの使い分け (Fetch、Firecrawl、Markdownify)"
emoji: "🐸"
type: "tech"
topics: ["mcp", "windsurf", "ai", "生成ai", "ai駆動開発"]
published: true
---

## 💡 MCPの始め方シリーズについて

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。本記事では、ウェブコンテンツの取得や処理に関連する3つのMCPサーバー（Fetch、Firecrawl、Markdownify）の特徴と使い分けについて詳しく解説します。

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
11. 👉 [ウェブの情報を取得するMCPの使い分け (Fetch、Firecrawl、Markdownify)](./mcp-server-tutorial-reference-web-mcp)

---

## 📊 機能比較表

| 機能 | Fetch MCP | Firecrawl MCP | Markdownify MCP |
|------|-----------|---------------|------------------|
| 基本的なウェブページ取得 | ✅ シンプル | ✅ 高機能 | ✅ Markdown変換 |
| インストールの手軽さ | ✅ 非常に簡単 | ⚠️ やや複雑 | ⚠️ 依存関係あり |
| メモリ使用量 | ✅ 軽量 | ⚠️ やや重い | ⚠️ 中程度 |
| 複数ページのクロール | ❌ 非対応 | ✅ 対応 | ❌ 非対応 |
| 検索機能 | ❌ 非対応 | ✅ 対応 | ❌ 非対応 |
| ウェブページの分析 | ❌ 基本的な取得のみ | ✅ AI分析機能あり | ❌ 基本的な変換のみ |
| スクリーンショット | ❌ 非対応 | ✅ 対応 | ❌ 非対応 |
| Markdownに変換 | ✅ 基本的 | ✅ 高機能 | ✅ 専門 |
| PDFから変換 | ❌ 非対応 | ❌ 非対応 | ✅ 対応 |
| 画像/オーディオから変換 | ❌ 非対応 | ❌ 非対応 | ✅ 対応 |
| Officeファイルから変換 | ❌ 非対応 | ❌ 非対応 | ✅ 対応 |
| YouTube字幕取得 | ❌ 非対応 | ✅ 対応 | ✅ 対応 |
| ディープリサーチ | ❌ 非対応 | ✅ 対応 | ❌ 非対応 |

## 🔍 ウェブコンテンツ関連の3つのMCPサーバー

### Fetch MCP

[Fetch MCP Server](./mcp-server-tutorial-11-fetch)は、Model Context Protocolの公式リファレンス実装の一つで、シンプルで軽量な設計が特徴です。単一ページの情報取得に特化しており、基本的なウェブコンテンツの取得と処理を効率的に行えます。

主な機能：

- ウェブページの内容をMarkdown形式で取得
- 長いウェブページを分割して読み込み（チャンク読み込み）
- 生のHTMLコンテンツの取得（オプション）
- 取得するコンテンツの長さ制限

### Firecrawl MCP

[Firecrawl MCP Server](./mcp-server-tutorial-08-firecrawl)は、複数ページのクロールや検索機能、高度な分析機能を備えた強力なMCPサーバーです。ウェブサイト全体をLLM用に変換するのに適しています。

主な機能：

- 複数ページのクロールと分析
- ウェブ検索と結果の分析
- スクリーンショットの取得
- ディープリサーチの実行
- YouTube字幕の取得
- AI分析機能

※ 真の Deep Research はまだ正式リリースはされていない (2025-03-11 現在)
[Deep Research - Firecrawl](https://www.firecrawl.dev/deep-research)

### Markdownify MCP

[Markdownify MCP Server](./mcp-server-tutorial-09-markdownfy)は、ウェブコンテンツだけでなく、様々なファイル形式をMarkdownに変換することに特化したMCPサーバーです。

主な機能：

- ウェブページをMarkdownに変換
- PDFをMarkdownに変換
- Office文書（DOCX、XLSX、PPTX）をMarkdownに変換
- 画像をMarkdownに変換（メタデータ付き）
- オーディオをMarkdownに変換（文字起こし付き）
- YouTube動画の字幕をMarkdownに変換

## 🔄 おすすめの使い分け

- **Fetch MCP**: 単一ページの情報取得、軽量な環境での使用、シンプルな使い方を好む場合。公式リファレンス実装なので安定性が高い。
- **Firecrawl MCP**: 複数ページのクロール、検索機能、高度な分析、ディープリサーチが必要な場合。ウェブサイト全体をLLM用に変換するのに適している。
- **Markdownify MCP**: ウェブページやPDF、Officeファイル、画像、オーディオを構造化されたMarkdownに変換して保存したい場合。多様なファイル形式の変換に特化している。

## 📋 具体的なシナリオ別の使い分け

- **単純な情報参照**: Fetch MCP
  → 特定のウェブページを素早く参照したい場合

- **複数ページの調査研究**: Firecrawl MCP
  → トピックに関する広範な情報収集や分析が必要な場合

- **コンテンツの整理と保存**: Markdownify MCP
  → ウェブページやドキュメントを構造化して保存したい場合

- **APIドキュメントの参照**: Fetch MCP
  → 特定のAPIドキュメントを参照したい場合

- **ニュース記事の要約作成**: Fetch MCP + Markdownify MCP
  → ニュース記事を取得して構造化した要約を作成したい場合

- **複数ソースの比較分析**: Firecrawl MCP
  → 複数の情報ソースを比較して分析したい場合

- **PDFやOfficeファイルの変換**: Markdownify MCP
  → 様々なファイル形式をMarkdownに変換したい場合

これらのサーバーは相互に補完し合う関係にあり、状況に応じて適切なサーバーを選択することで、より効率的な情報収集と整理が可能になります。

## 🔄 組み合わせ活用例

### Fetch MCP + Markdownify MCP

単一ページの情報取得と、その情報の構造化・保存を組み合わせることで、重要な情報を効率的に整理できます。

### Firecrawl MCP + Markdownify MCP

複数ページから収集した情報を、Markdownifyを使って様々な形式で保存・整理することができます。

### 全てのMCPを状況に応じて使い分け

プロジェクトの性質や情報収集の目的に応じて、最適なMCPサーバーを選択することで、AIとの情報収集作業を最大限に効率化できます。

## 💬 Claude Desktopのカスタムインストラクション例

**※現在試行錯誤中、アップデートする可能性が高いです。**

Claude Desktopでウェブコンテンツ取得のMCPを使い分けるためのカスタムインストラクション（システムプロンプト）の例です。これをカスタムインストラクションに追加することで、Claudeはあなたの優先順位に従ってMCPサーバーを選択するようになります。

```markdown
## ウェブコンテンツ取得のMCP使い分け

ウェブからの情報取得が必要な場合は、以下の優先順位で対応してください：

1. 基本的には「Fetch MCP Server」を優先して使用してください。
   - 単一ページの情報取得
   - 軽量で高速な動作が必要な場合
   - APIドキュメントの参照

2. 以下の場合のみ「Firecrawl MCP」を使用してください：
   - 複数ページにまたがる情報収集が必要な場合
   - ウェブサイト全体のクローリングが必要な場合
   - 検索機能を使った情報の絞り込みが必要な場合
   - ディープリサーチが必要な場合
   - YouTube字幕の取得が必要な場合

3. 以下の場合のみ「Markdownify MCP」を使用してください：
   - PDF、Office文書、画像、オーディオなどの変換が必要な場合
   - コンテンツの高度な構造化と再構成が必要な場合
   - YouTube字幕の取得が必要な場合

特に指示がない限り、常に「Fetch MCP Server」を最初に試してください。他のMCPサーバーは、Fetch MCPで対応できない特殊なケースでのみ使用してください。
```

このカスタムインストラクションを追加することで、Claudeはウェブコンテンツの取得時に基本的にはFetch MCPを優先して使うようになります。そして、特定の機能が必要な場合にのみ、他の2つのMCPサーバーを使い分けるようになります。

---

## 📚 関連記事

- [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
- [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
- [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
- [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
