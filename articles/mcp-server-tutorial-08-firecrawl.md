---
title: "【MCPのトリセツ #8】Firecrawl MCP：スクレイピングでウェブ情報を取得・分析"
emoji: "🐸"
type: "tech"
topics: ["mcp", "firecrawl", "claude", "windsurf", "ai", "生成ai", "ai駆動開発"]
published: true
---

## 💡 MCPの始め方シリーズについて

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。今回は、Firecrawl MCPの導入方法と活用テクニックを紹介します。リサーチや情報収集の効率化を可能にします！

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. [Slack MCPサーバー：チームコミュニケーションを強化](./mcp-server-tutorial-07-slack)
8. 👉 [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
9. [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)

---

## 🚀 Firecrawl MCPでできること

「AI にウェブサイトの内容を分析してもらいたい...」
「複数のページをクロールして情報を集めてきてほしい...」
「競合サイトのデザインや機能を調査したい...」

Firecrawl MCPサーバーを使えば、AIがウェブサイトの内容を詳細に理解し、複数ページのクロール、情報の抽出、検索、分析などを行えるようになります。

- **スクレイピング**: JavaScriptレンダリングによる高度なウェブスクレイピング
- **クロール**: 特定のサイトを起点に複数ページを自動探索
- **検索**: ウェブ上の情報をキーワードで検索
- **情報抽出**: 特定の要素やパターンに基づく構造化データの抽出
- **バッチ処理**: 複数URLの同時処理
- **ディープリサーチ**: 複数の情報源を組み合わせた詳細な調査

さらに、以下のような高度な機能も備えています：

- モバイル/デスクトップのビューポート切り替え
- タグインクルード/エクスクルードによるコンテンツフィルタリング
- カスタムアクション（クリック、スクロールなど）の実行
- 指数バックオフによる自動再試行
- クレジット使用量の監視

## 🏷️ 料金プラン

Firecrawl は無料枠もありますが、有料のツールです。
クレジットが不足した場合は、$9/月で1,000クレジットを追加購入できます。
詳細は[公式サイト](https://www.firecrawl.dev/pricing)を参照してください。

- **Free Plan**
  - 無料（一回限り）、500クレジット
  - 500ページのスクレイピング相当
  - 毎分10スクレイプ、1クロール

- **Hobby Plan**
  - $16/月、3,000クレジット/月
  - 3,000ページのスクレイピング相当
  - 毎分20スクレイプ、3クロール
  - 1シート

- **Standard Plan**
  - $83/月、100,000クレジット/月
  - 100,000ページのスクレイピング相当
  - 毎分100スクレイプ、10クロール
  - 3シート、標準サポート

---

## 🛠️ セットアップ手順

Firecrawl MCPサーバーの導入は以下のステップで行います。

### 1. Firecrawl APIキーの取得

[Firecrawl公式サイト](https://www.firecrawl.dev/)でアカウントを作成し、APIキーを取得します。
> 🔥 [このリンク](https://www.firecrawl.dev/referral?rid=W385F95R)から登録すると、10万トークン + 1,000クレジットが無料で付与されます。

### 2. MCPサーバーの設定

取得したAPIキーを使って、Firecrawl MCPサーバーを設定します。  
(※ mcp-installer でのインストールは、私は失敗しました。)

#### Claude Desktop の場合

`~/Library/Application Support/Claude/claude_desktop_config.json` を開き、以下の設定を追加します。

```json
{
  "mcpServers": {
    "firecrawl-mcp": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "YOUR_API_KEY_HERE",
        "FIRECRAWL_RETRY_MAX_ATTEMPTS": "5",
        "FIRECRAWL_RETRY_INITIAL_DELAY": "2000",
        "FIRECRAWL_RETRY_MAX_DELAY": "30000",
        "FIRECRAWL_RETRY_BACKOFF_FACTOR": "3",
        "FIRECRAWL_CREDIT_WARNING_THRESHOLD": "2000",
        "FIRECRAWL_CREDIT_CRITICAL_THRESHOLD": "500"
      }
    }
  }
}
```

- `YOUR_API_KEY_HERE` に取得したAPIキーをセット
- 環境設定オプション
  - 再試行構成
    - `FIRECRAWL_RETRY_MAX_ATTEMPTS = 5`: 最大再試行回数
    - `FIRECRAWL_RETRY_INITIAL_DELAY = 2000`: 初回遅延（ミリ秒）
    - `FIRECRAWL_RETRY_MAX_DELAY = 30000`: 最大遅延（ミリ秒）
    - `FIRECRAWL_RETRY_BACKOFF_FACTOR = 3`: バックオフ係数
  - クレジット監視
    - `FIRECRAWL_CREDIT_WARNING_THRESHOLD = 2000`: 警告しきい値
    - `FIRECRAWL_CREDIT_CRITICAL_THRESHOLD = 500`: 危険しきい値

#### Windsurf の場合

`~/.codeium/windsurf/mcp_config.json` を開き、同様の設定を追加します。

---

## 👨‍💻 基本的な使い方 (プロンプト)

### 指定したURLのウェブサイトをスクレイピングし内容を分析

```
このウェブサイトの情報を取得して分析して
https://example.com
```

### 複数ページのクロール

```
https://example.com
このウェブサイトを深さ2レベルまでクロールして、主要なコンテンツと構造を分析して
```

### キーワード検索 (関連情報を収集して要約)

```
「クラウドネイティブアプリケーション開発」について最新の情報を検索して、主要なトレンドと技術をまとめて
```

### 構造化データの抽出し表形式でまとめる

```
このECサイトから、製品名、価格、説明を抽出して表形式でまとめて
https://example-shop.com/products
```

## 💡 活用テクニック

### 1. 競合サイトを分析し、比較レポートを作成

```
https://competitor1.com
https://competitor2.com
https://competitor3.com
これらのウェブサイトを分析して、以下を比較してください

- UI/UXデザインの特徴
- 主要な機能と特長
- コンテンツ戦略
- ターゲットユーザー層
```

### 2. クロールし、指定した要素を抽出 (スクレイピング)

```
ウェブサイト https://example.com をクロールして、以下の情報を収集して：
- クロール深度: 2レベル
- サブドメインは含めない
- 出力形式: JSON
- 抽出する要素:
  - タイトル: head > title セレクタから
  - コンテンツ: .article-body セレクタから
  - 著者: .author-name セレクタから
  - 公開日: .publish-date セレクタから
この情報を使って、ウェブサイトの構造とコンテンツを分析し、主要な記事や情報を抽出して。
```

### 3. ディープリサーチ (複数の情報源を組み合わせての詳細な調査)

```
「量子コンピューティングの最新応用例」についてディープリサーチを実行してください。
科学論文のプレプリントサーバー、研究機関のウェブサイト、技術ブログなど幅広い情報源から情報を集めてください。
```

### 4. 特定のタグやセクションを抽出し整理

```
https://example.com/blog のすべての記事から、<code>タグで囲まれたコード例だけを抽出し、言語別に分類してください
```

## 🧩 他のMCPサーバーとの組み合わせ

### コンテンツを取得し、ローカルファイルとして保存 (Firecrawl MCP + Filesystem MCP)

```
1. https://documentation.example.com サイトから全てのチュートリアルページを取得して
2. 各チュートリアルの内容をMarkdown形式に変換して
3. /Users/username/Documents/tutorials/ フォルダに保存して
```

### ウェブ上の情報を分析し、結果をGitHubリポジトリに保存 (Firecrawl MCP + GitHub MCP)

```
1. 「Rust WebAssembly チュートリアル」をテーマにウェブ検索を実行し、最も良質な情報を集めて
2. 集めた情報を元に、初心者向けのチュートリアルMarkdownファイルを作成して
3. GitHub上の「my-tutorials」リポジトリに「rust-wasm-guide.md」として保存して
```

### 検索した YouTube動画の内容を分析し加工 (Firecrawl MCP + YouTube MCP)

```
1. 「機械学習 初心者」をテーマにした最新のYouTubeチュートリアルを探して
2. 各動画の内容を要約して
3. 最も分かりやすいチュートリアルランキングを作成して
```

## ⚠️ Firecrawl MCPを使用する際の注意点

1. **クレジット消費**: 大量のページをスクレイピングやクロールするとクレジットを消費します。設定したクレジット警告しきい値を活用しましょう。

2. **使用ポリシー**: ウェブサイトの利用規約に従いましょう。過度なスクレイピングは規約違反になる場合があります。

3. **レート制限**: 短時間に多数のリクエストを送ると、サイト側でブロックされる可能性があります。

4. **JavaScript依存**: 一部のサイトはJavaScriptに依存したコンテンツを持ちます。正確に取得するにはレンダリングオプションを使用しましょう。

5. **データ品質**: 自動抽出したデータは常に検証が必要です。特にフォーマットが複雑なサイトでは注意が必要です。

## 📝 まとめ

Firecrawl MCPサーバーは、AIによるウェブ情報の取得・分析を強力に支援するツールです。スクレイピング、クロール、検索、情報抽出など多彩な機能により、リサーチや情報収集が大幅に効率化されます。

市場調査、競合分析、トレンド把握、データ収集など、さまざまな用途に活用できるでしょう。他のMCPサーバーと組み合わせることで、情報収集から加工、保存までの一連のワークフローを自動化することも可能です。

ぜひFirecrawl MCPを導入して、AIのウェブ情報活用能力を拡張してみてください！

## 📚 参考リンク

- [Firecrawl公式サイト](https://www.firecrawl.dev/)
- [firecrawl-mcp GitHub リポジトリ](https://github.com/mendableai/firecrawl-mcp-server)
- [Firecrawl API ドキュメント](https://docs.firecrawl.dev/)

次回以降の記事でも、便利なMCPサーバーを紹介していきます。お楽しみに！

## 🙏 さいごにおねだり

この記事が役に立ったと思ったら、🔥 [このリンク](https://www.firecrawl.dev/referral?rid=W385F95R)から Firecrawl に登録してください。登録する方にも10万トークン + 1,000クレジットが無料で付与されます。
