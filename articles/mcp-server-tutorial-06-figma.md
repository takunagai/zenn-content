---
title: "【MCPのトリセツ #6】Figma MCP：デザインとコードを効率的に連携"
emoji: "🐸"
type: "tech"
topics: ["mcp", "figma", "claude", "windsurf", "ai", "生成ai", "ai駆動開発"]
published: true
---

## 💡 MCPの始め方シリーズについて

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。今回は、Figma MCP の導入方法と活用例を解説します。デザイナーとエンジニアの連携が格段にスムーズになり、開発効率が向上するはずです。

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. 👉 [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. [Slack MCPサーバー：チームコミュニケーションを強化](./mcp-server-tutorial-07-slack)
8. [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
9. [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)
11. [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
12. [Blender MCP Server: 会話で Blender を操作し3Dモデルを作成](./mcp-server-tutorial-12-blender)
13. [ウェブの情報を取得するMCPの使い分け (Fetch、Firecrawl、Markdownify)](./mcp-server-tutorial-reference-web-mcp)

---

## 🧩 Figma MCPとは？何ができるの？

「Figma で作られた UI要素をコード化するのに時間がかかる...」
「Figma のデザイン仕様を正確に反映したコードを書きたい...」
「カラーコードやサイズを毎回スクリーンショットから確認するのが面倒...」

Figma MCPサーバーを使えば、こんな悩みを減らせます。AI が Figma のデザインデータに直接アクセスし、そのデータを理解・操作するための橋渡しをします。結果、正確な情報を元にコード変換や実装アドバイスを行えるようになります。

1. **正確なデザイン情報の取得**：色、サイズ、フォント、間隔などの正確な数値を取得
2. **コンポーネントデータの抽出**：ボタン、カード、ナビゲーションなどのコンポーネント情報を分析
3. **デザインからコードに変換**：Figmaデザインを対応するHTML/CSS/JavaScriptやReactコンポーネントに変換
4. **デザイントークンの抽出**：カラーパレット、タイポグラフィ、スペーシングなどのデザイントークンを一括取得
5. **レスポンシブデザインの分析**：異なるブレークポイントでのレイアウト変化を理解
6. **デザインコメントの参照**：Figma ファイル内のコメントやフィードバックを参照しながら実装

## 🛠️ セットアップ手順

Figma MCPサーバーのセットアップは3つのステップで完了します。

### 1. Figma APIキーの取得

まず、Figma の API にアクセスするためのパーソナルアクセストークンを取得します。

1. [Figma](https://www.figma.com)にログイン
2. 右上のアイコンから「Settings」を選択
3. 左側メニューから「Account」タブを選択
4. 下部の「Personal access tokens」セクションで「Create a new personal access token」をクリック
5. 適当なトークンの名前（例：`figma-mcp-2025-03-01`）を入力し、「Create token」をクリック
6. 表示されたAPIキーをコピー（**注意**: このキーは一度しか表示されないので安全な場所に保存）

### 2. エディタの設定

Claude Desktop や AIエディタ (Windsurf や Cursor) が「Figma MCP」を利用できるように設定ファイルを編集します。

#### Claude Desktopの場合

Claude Desktopの設定ファイル（`~/Library/Application Support/Claude/claude_desktop_config.json`）を開く（なければ作成）
2. 以下の内容を追加（既存の設定がある場合は `mcpServers` オブジェクト内に追記）

```json
{
  "mcpServers": {
    "figma-developer-mcp": {
      "command": "npx",
      "args": ["-y", "figma-developer-mcp", "--stdio"],
      "env": {
        "FIGMA_API_KEY": "your_figma_api_key_here"
      }
    }
  }
}
```

`your_figma_api_key_here` の部分を、先ほど取得したFigma APIキーに置き換えてください。

#### Windsurf の場合

1. `~/.codeium/windsurf/mcp_config.json` ファイルを開き、同様の設定を追加します。

### 3. Figma MCPサーバーの起動

アプリケーションを再起動すると自動的にMCPサーバーが起動します。  
ターミナルで手動で起動する場合は、以下のコマンドで MCPサーバーを起動します：

```bash
npx figma-developer-mcp --figma-api-key=your_figma_api_key_here
```

## 👨‍💻 基本的な使い方 (プロンプト)

Figma MCPサーバーの基本的な使い方を紹介します。

### Figma ファイルの接続

```text
Figma MCPサーバーに繋いで https://www.figma.com/file/xxxxXXXXxxxxXXXX/ProjectName
```

AIがFigmaファイルに接続し、デザインデータにアクセスできるようになります。

### デザイン要素の取得

```text
このFigmaファイルのメインボタンのスタイル情報を教えて
```

AIはボタンの色、サイズ、フォント、角丸などの正確な情報を取得して表示します。

### コード生成

```text
このボタンをReactコンポーネントとして実装して
```

AIは取得したデザイン情報を元に、対応するReactコンポーネントのコードを生成します。

### レスポンシブ対応

```text
このコンポーネントをモバイル対応にするには？
```

AIはデザインのブレークポイントを分析し、レスポンシブ対応のコードを提案します。

## 🔍 特定のエリアやコンポーネントの取得方法

大規模なFigmaファイルから特定の要素だけを取得するには、ノードID（node-id）を使用すると効率的です：

### ノードIDの取得方法

1. **Figmaデザインファイルを開く**
2. **対象の要素を選択する**（例：「ホーム」フレーム内の「Hero」セクション）
3. **ノードIDの確認方法**：
   - 要素を選択した状態でブラウザのアドレスバーを確認。URLに `node-id=X-Y` という形式で表示される
   - または、要素を右クリックして「Copy/Paste as」→「Copy link」を選択

### AIエディタでの活用例

```text
Figma MCPでファイルの node-id=123-456 のコンポーネント情報を取得して
```

または

```text
Figma MCPでこのURL（https://www.figma.com/file/...?node-id=789-012）の「Hero」セクションの実装コードを生成して
```

## 💡 実践的な活用例

Figma MCPをより実践的に活用する方法をいくつか紹介します。

### 1. デザインシステムを分析し、変数として整理

```text
Figmaコンポーネントライブラリからカラー、タイポグラフィ、スペーシングなどのデザイントークンを抽出し、CSS変数として出力して
```

### 2. デザイン仕様をローカルファイルとして保存 (Figma MCP + Filesystem MCP)

```text
1. Figmaファイル（https://www.figma.com/file/xxxxx）のデザイントークンを分析して
2. カラー、タイポグラフィ、スペーシングの情報をCSS変数として /Users/yourname/Projects/design-system/tokens.css ファイルに保存して
```

### 3. バリエーションやバリアントを含むコンポーネントを生成

```text
このボタンの4つの状態（通常、ホバー、アクティブ、無効）を含む Reactコンポーネントセットを生成して
```

```text
このボタンコンポーネントを分析して、class-variance-authority（CVA）を使用したバリアント（サイズ: sm, md, lg、バリエーション: primary, secondary, outline, ghost）を持つ Tailwind CSS のボタンコンポーネントを実装して
```

### 4. Figmaのプロトタイプ情報を元に、アニメーションの実装コードを提案

```text
このモーダルのトランジションアニメーションを Framer Motion で実装するには？
```

### 5. デザイン仕様書の自動生成

```text
このヘッダーコンポーネントの詳細な仕様書（サイズ、色、間隔、フォントなど）をMarkdownで作成して
```

### 6. デザインの一貫性チェック

```text
このデザインのカラーパレットとタイポグラフィの使用を分析し、デザインシステムから外れている部分があれば指摘して
```

## 📝 まとめ

Figma MCPを導入することで、デザインとコーディングの橋渡しがスムーズになり、開発効率が向上します。デザイナーの意図を正確に反映したコード実装が容易になり、デザインの一貫性も保ちやすくなります。

特にフロントエンド開発者や UI/UXデザイナーにとって、Figma MCPは強力なアシスタントとなるでしょう。デザインデータを直接取得できることで、情報の見落としや解釈の誤りを減らし、より質の高い実装が可能になります。

ぜひFigma MCPを活用して、デザインとコードの連携を効率化してみてください！

新しいMCP記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。フォローとツッコミ、お待ちしています！

## 📚 参考リンク

- [Figma API公式ドキュメント](https://www.figma.com/developers/api)
- [figma-developer-mcp GitHub リポジトリ](https://github.com/figma/figma-developer-mcp)
- [Figma開発者コミュニティ](https://www.figma.com/community)
- [AIエディタ：Windsurf公式ドキュメント](https://codeium.com/windsurf)
- [AIエディタ：Cursor公式サイト](https://cursor.sh/)

次回の記事では、Slackでの投稿や情報の取得ができる「[Slack MCPサーバー](./mcp-server-tutorial-07-slack)」について解説します。お楽しみに！
