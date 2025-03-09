---
title: "【MCPのトリセツ #3】YouTube MCPサーバー：動画の内容を取得"
emoji: "🐸"
type: "tech"
topics: ["mcp", "youtube", "github", "claude", "windsurf", "ai", "生成ai", "ai駆動開発"]
published: true
---

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。今回は YouTube MCPサーバーを導入し、AIにYouTube動画の内容を理解させ、要約や分析を依頼できるようにする手順を紹介します。

YouTube MCPサーバーの導入方法と活用テクニックを紹介します。

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. 👉 [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. [Slack MCPサーバー：チームコミュニケーションを強化](./mcp-server-tutorial-07-slack)
8. [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
9. [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)

---

## ✨ YouTube MCPサーバー

YouTube の長い動画を視聴するのは時間がかかりますし、見ても時間が経つと忘れてしまいがちですよね。

動画を字幕テキストとして取得し、AIに分析させることで、情報収集が格段に効率化されますし、情報をまとめての永続化も可能になります。

## 🚀 YouTube MCPでできること

YouTube MCPサーバーを導入すると、次のようなことが可能になります：

- YouTube動画の内容を要約してもらう
- 長い講義や解説動画のポイントを抽出
- 英語の動画を日本語で要約してもらう
- 複数の動画を比較分析してもらう
- 特定のトピックに関する情報だけを抽出

つまり、膨大な動画コンテンツを効率的に消化し、必要な情報だけを取り出せるようになるのです。

---

## 👨‍💻 使用例（プロンプト）

YouTube MCP を使うのは超カンタン。以下のようなプロンプトで機能します。  
「YouTubeを」と明示的に指定しなくても、URLを認識して適切に処理してくれます。

### 基本的な活用プロンプト

#### 📌 シンプルな動画要約

```
要約して
https://www.youtube.com/watch?v=oXq7trXF4aI
```

#### 📌 翻訳付き要約

```
要約して。もし英語なら要約する前に日本語に翻訳して
https://www.youtube.com/watch?v=oXq7trXF4aI
```

#### 📌 主要ポイントの抽出

```
この動画の主要なポイントを5つにまとめて
https://www.youtube.com/watch?v=oXq7trXF4aI
```

#### 📌 情報の階層化

```
この動画について、以下の形式で情報を整理して
1. 3行の概要
2. 主要なポイント（箇条書き）
3. 詳細な説明が必要な部分
https://www.youtube.com/watch?v=xxxxx
```

#### 📌 特定トピックの情報抽出

```
「Windsurf」のトピックに関する情報だけを抽出して
https://www.youtube.com/watch?v=xxxxx
```

#### 📌 複数動画の比較分析

```
以下の動画を比較して、共通点と相違点を教えて
https://www.youtube.com/watch?v=oXq7trXF4aI
https://www.youtube.com/watch?v=yyyyy
```

### 教育・学習向けプロンプト

#### 📌 学習ノート作成

```
この講義動画の内容を学習ノートの形式でまとめて。重要な概念には太字を使い、例や応用方法も含めて
https://www.youtube.com/watch?v=xxxxx
```

#### 📌 練習問題生成

```
この動画の内容を基に、5つの練習問題と解答を作成して
https://www.youtube.com/watch?v=xxxxx
```

### ビジネス・会議向けプロンプト

#### 📌 会議内容の構造化

```
この会議動画から、決定事項、アクションアイテム、締め切り日をリストアップして
https://www.youtube.com/watch?v=xxxxx
```

#### 📌 製品情報の整理

```
この製品発表の動画から、新機能、価格、発売日、競合製品との比較点を表形式でまとめて
https://www.youtube.com/watch?v=xxxxx
```

### コンテンツ制作・分析向けプロンプト

#### 📌 スクリプトから記事作成

```
この動画のスクリプトを抽出し、話し言葉を書き言葉に変換して読みやすい記事形式にして
https://www.youtube.com/watch?v=xxxxx
```

#### 📌 視聴者関心分析

```
この動画の内容を分析して、視聴者が最も関心を持ちそうな3つのトピックと、それぞれについての詳細な解説を提供して
https://www.youtube.com/watch?v=xxxxx
```

### 多言語・翻訳活用プロンプト

#### 📌 専門用語付き翻訳要約

```
この英語の講演動画を、重要ポイントを逃さずに日本語で要約し、専門用語には英語の原語も括弧内に記載して
https://www.youtube.com/watch?v=xxxxx
```

#### 📌 初心者向け解説

```
この動画の内容を初心者にもわかりやすく説明して。専門用語があれば、簡単な言葉で解説も加えて
https://www.youtube.com/watch?v=xxxxx
```

### 複数動画の高度な分析プロンプト

#### 📌 複数動画の比較レポート

```
以下の2つの動画を比較して、共通点と相違点を教えて
https://www.youtube.com/watch?v=xxxxx
https://www.youtube.com/watch?v=yyyyy
```

```
これら3つの動画から、共通するテーマと各動画独自の視点を比較した分析レポートを作成して
https://www.youtube.com/watch?v=xxxxx
https://www.youtube.com/watch?v=yyyyy
https://www.youtube.com/watch?v=zzzzz
```

#### 📌 対立意見の客観分析

```
この2つの対立する意見の動画を分析し、両者の主張、根拠、論理的な強み・弱みを客観的に評価して
https://www.youtube.com/watch?v=xxxxx
https://www.youtube.com/watch?v=yyyyy
```

### 実用的な応用プロンプト

#### 📌 レシピ抽出

```
この料理レシピ動画から、材料リスト、調理手順、調理時間、難易度を抽出して整理して
https://www.youtube.com/watch?v=xxxxx
```

#### 📌 技術チュートリアルの手順書作成

```
この技術チュートリアル動画から、ステップバイステップの手順書を作成して。各ステップにはコマンドやコードも含めて
https://www.youtube.com/watch?v=xxxxx
```

---

## 🛠️ インストール手順

YouTube MCPサーバーをインストールするには、2つのステップが必要です：

### 1. 依存ライブラリ YT-DLP のインストール

まず、YouTubeから字幕をダウンロードするための [yt-dlp](https://github.com/yt-dlp/yt-dlp) をインストールします。これは1000以上のサイトに対応したオープンソースの動画・音声ダウンローダーです。

**macOSの場合：**

```bash
brew install yt-dlp
```

**Windowsの場合：**

```
winget install yt-dlp
```

インストール後、確認のためコマンドラインで以下を実行：

```bash
yt-dlp --version
```

バージョン番号が表示されれば成功です。

### 2. YouTube MCPサーバーのインストール

下記のいずれかの方法でインストールします。

#### 方法1： mcp-installer を使う場合（推奨）

前回または[シリーズ記事#0](./mcp-server-tutorial-01-install.md)でmcp-installerをセットアップ済みであれば、Claude に以下のように指示するだけでインストールできます。 (※「mcp-youtube」ではなく「@anaisbetts/mcp-youtube」と指定する必要があります。)

```
MCPサーバー @anaisbetts/mcp-youtube をインストールして
```

#### 方法2：設定ファイルを直接編集する場合

Claude Desktop の設定ファイル（`~/Library/Application Support/Claude/claude_desktop_config.json`）を開き、以下の設定を追加します。既に他のMCPサーバーを設定している場合は、"mcpServers" オブジェクト内に追加してください。

```json
{
  "mcpServers": {
    "youtube": {
      "command": "npx",
      "args": [
        "-y",
        "@anaisbetts/mcp-youtube"
      ]
    }
  }
}
```

設定後、Claude Desktop を再起動。

## 💡 活用テクニック: Filesystem MCP との連携

[Filesystem MCP](./mcp-server-tutorial-02-filesystem.md)と組み合わせると、動画の内容をローカルファイルに直接保存できます。

```
この動画を要約し、ドキュメントフォルダ/video-summary.md に保存してください：
https://www.youtube.com/watch?v=xxxxx
```

## ⚠️ YouTube MCPサーバーを使用する際の注意点と制限事項

1. **字幕の有無**: 字幕が提供されていない動画は処理できません
2. **視覚情報**: 画像や映像そのものは理解できず、字幕のテキストのみを処理します
3. **長い動画**: 非常に長い動画は処理に時間がかかったり、トークンの制限にかかる場合があります
4. **多言語対応**: 英語以外の言語でも動作しますが、精度は言語によって異なります

## 📝 まとめ

YouTube MCPサーバーを導入することで、膨大な動画コンテンツから効率的に情報を抽出し、時間を節約できます。技術的な解説動画、講義、プレゼンテーションなど、様々な教育コンテンツをテキストとして理解し、要約や分析が可能になります。

学習効率の向上や情報収集のスピードアップに、ぜひYouTube MCPサーバーを活用してみてください！

新しいMCP記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。フォローとツッコミ、お待ちしています！

## 📚 参考リンク

- [yt-dlp公式GitHub](https://github.com/yt-dlp/yt-dlp)
- [Homebrew yt-dlp](https://formulae.brew.sh/formula/yt-dlp)
- [MCP YouTube サーバー](https://github.com/anaisbetts/mcp-youtube)
- [MCP初心者ガイド動画](https://www.youtube.com/watch?v=Y_kaQmhGmZk)

次回の記事では、MarkdownやWordなど様々な形式のドキュメント形式を変換できる「[mcp-pandoc](./mcp-server-tutorial-04-pandoc)」について解説します。お楽しみに！
