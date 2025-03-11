---
title: "【MCPのトリセツ #11】Fetch MCP Server: ウェブコンテンツを取得・処理"
emoji: "🐸"
type: "tech"
topics: ["mcp", "windsurf", "ai", "生成ai", "ai駆動開発", "fetch"]
published: true
---

## 💡 MCPの始め方シリーズについて

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。今回は、Fetch MCPサーバーの導入方法と活用テクニックを紹介します。AIとの会話でウェブページの内容を取得・処理できるようになり、情報収集が格段に効率化します！

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
11. 👉 [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
11. [ウェブの情報を取得するMCPの使い分け (Fetch、Firecrawl、Markdownify)](./mcp-server-tutorial-reference-web-mcp)

---

[servers/src/fetch - modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch)

## 🚀 Fetch MCP Server でできること

Fetch MCP Serverは、ウェブコンテンツの取得と処理に特化したMCPサーバーです。以下のことができます：

- ウェブページの内容をMarkdown形式で取得
- 長いウェブページを分割して読み込み（チャンク読み込み）
- 生のHTMLコンテンツの取得（オプション）
- 取得するコンテンツの長さ制限

Fetch MCPは、Model Context Protocolの公式リファレンス実装の一つで、シンプルで軽量な設計になっています。基本的なウェブコンテンツ取得に特化しており、単一ページの情報取得を効率的に行えます。

> 📝 **ノート**: Fetch MCP、Firecrawl MCP、Markdownify MCPの比較と使い分けについては、[ウェブの情報を取得するMCPの使い分け](./mcp-server-tutorial-reference-web-mcp)を参照してください。

---

## 👨‍💻 Fetch MCP Server プロンプトのサンプル

### 基本的なウェブページ取得

```text
https://example.com のコンテンツを取得して
```

```text
Python公式サイトの内容を要約して
```

### 長いコンテンツを分割して読む

```text
https://docs.python.org/3/tutorial/index.html の内容を取得して、長い場合は続きを読みたい
```

```text
前のページの続きを5000文字分読んで
```

### 特定のドキュメントから情報を抽出

```text
https://github.com/modelcontextprotocol/servers のREADMEから、MCPサーバーの一覧を抽出して
```

### 生のHTMLコンテンツを取得

```text
https://example.com の生のHTMLを取得して
```

### 複数ページの情報を組み合わせる

```text
PythonとJavaScriptの公式ドキュメントから、それぞれの配列操作の違いを比較して
```

### 技術文書の特定部分を参照

```text
Reactの公式ドキュメントからフックの使い方について説明して
```

### ニュースサイトから最新情報を取得

```text
BBCニュースの最新記事を取得して要約して
```

### APIドキュメントの参照

```text
OpenAI APIのドキュメントからChat Completions APIの使い方を教えて
```

### 高度なプロンプト例

#### 特定の情報をピンポイントで抽出

```text
https://nodejs.org/en/docs/ から最新のNode.jsバージョンとその主な機能を抽出して
```

#### 技術比較のための情報収集

```text
ReactとVueの公式ドキュメントからコンポーネントライフサイクルの違いを比較して表にまとめて
```

#### 複数ソースからの情報統合

```text
TensorFlowとPyTorchの公式ドキュメントから、初心者向けの機械学習モデル構築手順を比較して説明して
```

#### 特定のコード例を探す

```text
MDNのJavaScriptドキュメントから、非同期処理のベストプラクティスとコード例を探して
```

#### 最新の技術トレンド調査

```text
GitHubブログから最近の開発者向け新機能について情報を集めて要約して
```

---

## 🛠️ インストールと設定

Fetch MCP Serverは、Python製のMCPサーバーです。インストールは非常に簡単です。
uv がインストール済みなのが前提です (インストール方法はシリーズ初回を参照)。

### Claude デスクトップ側の設定

Claude Desktopの設定ファイル（claude_desktop_config.json）に以下を追加します：

```json
"mcpServers": {
  "fetch": {
    "command": "uvx",
    "args": ["mcp-server-fetch"]
  }
}
```

### オプション

####

- `args` に `--ignore-robots-txt` を追加で robots.txt を無視 (ユーザーからのリクエストのみ、モデル経由のリクエストは無視しない)
- `args` に `--user-agent=YourUserAgent` を追加で User-agent をカスタマイズ
  - 例：iPhone SE3 の場合：`--user-agent="Mozilla/5.0 (iPhone; CPU iPhone OS 16_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/16.0 Mobile/15E148 Safari/605.1"`

**メモ**：デフォルトの User-agent

- `ModelContextProtocol/1.0 (Autonomous; +https://github.com/modelcontextprotocol/servers)`
- `ModelContextProtocol/1.0 (User-Specified; +https://github.com/modelcontextprotocol/servers)`

---

## 📝 まとめ

Fetch MCP Serverは、ウェブコンテンツを取得・処理するシンプルで強力なツールです。以下のようなメリットが得られます：

- AIとの会話だけでウェブページの内容を簡単に取得可能
- Markdown形式での取得により、AIが内容を理解しやすい
- 長いコンテンツも分割して読み込めるチャンク機能
- 軽量で導入が簡単
- 他のMCPサーバーと組み合わせて使用可能

### 3つのMCPサーバーの実践的な使い分け

> 📝 **ノート**: 3つのMCPサーバーの実践的な使い分けや具体的なシナリオ別の比較については、[ウェブの情報を取得するMCPの使い分け](./mcp-server-tutorial-reference-web-mcp)を参照してください。

ぜひFetch MCP Serverを導入して、AIとのウェブ情報活用を効率化してみてください！

## 📚 参考リンク

- [Fetch MCP Server GitHub リポジトリ](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch)
- [Fetch MCP Server README](https://github.com/modelcontextprotocol/servers/blob/main/src/fetch/README.md)
- [Model Context Protocol 公式サイト](https://modelcontextprotocol.ai/)
