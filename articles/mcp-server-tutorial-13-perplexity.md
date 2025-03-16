---
title: "【MCPのトリセツ #13】Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行"
emoji: "🐸"
type: "tech"
topics: ["mcp", "windsurf", "ai", "ai駆動開発", "perplexity"]
published: true
---

## 💡 MCPの始め方シリーズについて

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。今回は、Perplexity MCPサーバーの導入方法と活用テクニックを紹介します。Perplexity MCPは、Perplexity の検索機能を活用してリアルタイムのWeb全体の調査結果から最も関連する洞察を返す Sonar API をAIとの会話から呼び出せるようにしてくれるコネクタです。

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
13. 👉 [Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行](./mcp-server-tutorial-13-perplexity)
14. [ウェブの情報を取得するMCPの使い分け (Fetch、Firecrawl、Markdownify)](./mcp-server-tutorial-reference-web-mcp)

---

- 利用には、有料でのみ利用可能の Perplexity の Sonar API Key が必要です
- Perplexity Pro プランには、毎月5ドル分の API 呼び出し枠が提供
- [ppl-ai/modelcontextprotocol - Github](https://github.com/ppl-ai/modelcontextprotocol)
- [Integrating MCP with Perplexity's Sonar API - Perplexity](https://docs.perplexity.ai/guides/mcp-server)
- [Sonar API 料金表/各モデルの説明 - Perplexity](https://docs.perplexity.ai/guides/pricing)

## 🚀 Perplexity MCP Server でできること

Perplexity MCP Serverは、Perplexity の Sonar API をAI との会話から呼び出すようにするコネクタです。

- リアルタイムのWeb全体の調査を実行し、最新情報を取得
- 複雑な検索クエリを自然言語でAIに依頼可能
- 検索結果を要約して整理された形で表示
- 検索結果から関連情報を抽出して回答を生成
- Web検索による情報収集プロセスを効率化
- Webの最新情報に基づいた正確な回答を提供
- 高度なChain of Thought（CoT）推論機能でより詳細な分析が可能（Reasoning）
- 複数情報源からの総合的な分析レポートを自動生成（Deep Research）
- 引用元を明示した信頼性の高い回答を提供

Perplexity MCPは、Perplexity 公式の MCP です。

---

## 👨‍💻 Perplexity MCP Server プロンプトのサンプル

### 基本的な検索クエリ

```text
日本の最新の経済指標について調べて、現在の経済状況を分析して
```

### 特定のトピックに関する詳細情報

```text
量子コンピューティングの最新の進展について、特に量子優位性の実証例を中心に詳しく教えて
```

### 技術比較

```text
React と Vue.js の最新バージョンの違いを比較し、それぞれの長所と短所をまとめて
```

### ニュース分析

```text
過去1週間の気候変動に関する主要ニュースをまとめ、各国の対応策を比較して
```

### 学術研究

```text
認知行動療法の最新研究について調査し、うつ病治療における効果の最新エビデンスを報告して
```

### 商品調査

```text
最新の高性能ノートパソコンを比較し、バッテリー寿命、性能、価格の観点から最もコスパの良い5つのモデルを選んで
```

### 統計・データ分析

```text
世界各国のワクチン接種率の最新データを収集し、地域ごとの傾向と相関要因を分析して
```

### 法的・規制情報

```text
EUのAI規制法の最新版について調査し、企業がコンプライアンスを確保するために必要な主要要件をまとめて
```

### 順序立てた構造化プロンプト

以下は、より複雑な調査や分析のための順序立てた構造化プロンプトの例です。これらのプロンプトでは、`[  ]`で囲まれた部分を具体的な内容に置き換えて使用してください。

#### ユースケース調査（sonar-pro向け）

```text
AIエージェントのユースケースを調査したいです。次のステップに従って調査結果を提供してください。

1. AIエージェントの主要な産業別ユースケースを10件程度、最新の情報源から調査してください
2. 各ユースケースについて以下を簡潔に説明してください：
   a. 産業/分野
   b. 具体的なユースケース
   c. 主なメリットと課題
3. 情報源を明記して、整理された形で結果を提示してください
```

#### 競合分析レポート（sonar-reasoning-pro向け）

```text
次のステップに従って、[製品名]と競合製品の比較分析を行ってください。

1. [製品名]の主要な競合製品を3〜5つ特定してください
2. 各製品について以下の情報を収集してください：
   a. 価格体系とプラン
   b. 主要機能と差別化ポイント
   c. ユーザーレビューと満足度評価
3. 収集した情報を基に、各競合製品の強みと弱みを分析してください
4. [製品名]が市場で優位性を確立するための戦略的推奨事項を提案してください
5. すべての情報源を引用して、整理された比較レポートを作成してください
```

#### 技術トレンド予測（sonar-deep-research向け）

```text
[技術分野]の今後3年間のトレンド予測を作成してください。以下の手順で進めてください：

1. 最新の研究論文、業界レポート、専門家の意見を調査し、[技術分野]における主要な技術的進歩を特定してください
2. これらの進歩が以下の領域にどのような影響を与えるか分析してください：
   a. 消費者行動と採用率
   b. ビジネスモデルと市場機会
   c. 規制環境と法的枠組み
3. 今後3年間の予測タイムラインを作成し、重要なマイルストーンを示してください
4. この分野に投資または注目している企業や投資家へのアクションアイテムを提案してください
5. すべての情報源を明記し、体系的にまとめた詳細レポートを作成してください
```

#### 学術文献レビュー（sonar-deep-research向け）

```text
[研究テーマ]に関する最新の学術研究の包括的なレビューを行ってください。以下のプロセスに従ってください：

1. 過去3年間に発表された[研究テーマ]に関する重要な学術論文を8〜10件特定してください
2. 各論文について以下を要約してください：
   a. 研究目的と主要な研究課題
   b. 使用された方法論とアプローチ
   c. 主な発見と結論
3. 特定された論文全体から浮かび上がる主要なテーマ、一致点、矛盾点を分析してください
4. この研究分野における現在の知識ギャップと、将来の研究の方向性を特定してください
5. 発見した知見を整理して提示し、すべての情報源を適切に引用してください
```

#### 市場調査と消費者インサイト（sonar-deep-research向け）

```text
[製品カテゴリー]の市場調査と消費者インサイトについて、詳細なレポートを作成してください。

1. [製品カテゴリー]の現在の市場規模、成長率、主要プレーヤーに関する最新データを収集してください
2. 消費者行動と購買意思決定に影響を与える主要な要因を特定してください
3. 最新の市場調査、消費者レビュー、ソーシャルメディアの分析から消費者の嗜好とトレンドを抽出してください
4. 地域間の違いや特定の人口統計グループにおける顕著なパターンを分析してください
5. 今後12〜24ヶ月の間に予想される市場変化と消費者行動の変化を予測してください
6. すべての情報源を明記し、データに基づいた実用的なビジネスインサイトを提供してください
```

---

## 🛠️ インストールと設定

Perplexity MCP Serverは、JavaScript製のMCPサーバーです。インストールは非常に簡単です。
npm がインストール済みなのが前提ということで解説を進めます。

### 🔑 Sonar API Keyの取得

Sonar とは、Perplexity の Sonar API です。以下の手順でAPI Keyを取得します：

1. [Perplexity公式サイト](https://docs.perplexity.ai/guides/getting-started)にアクセスし、アカウント登録をします
2. 開発者ダッシュボードから「APIキーの生成」を選択します
3. APIキーが発行されたら、安全な場所に保存します
4. このAPIキーは後ほど MCP Server の設定で使用します

なお、Sonar APIの利用には料金がかかります。Perplexity Proユーザーには毎月5ドル分の無料APIクレジットが提供されています。

### MCP Server の設定

MCPサーバーを置く場所に移動して、クローンします。  
(例：自分は ~/tools/mcp-server/ に各MCPサーバーを置いています)

```
git clone git@github.com:ppl-ai/modelcontextprotocol.git
```

modelcontextprotocol ディレクトリの中の perplexity-ask ディレクトリを MCPサーバー置き場に移動します。
(~/tools/mcp-server/perplexity-ask/ となるように)

中に移動して npm install を実行。

```
cd ~/tools/mcp-server/perplexity-ask/ && npm install
```

### Claude デスクトップ側の設定

Claude Desktopの設定ファイル（claude_desktop_config.json）に以下を追加します。
"YOUR_API_KEY_HERE" には、Sonar API Key を設定してください。

```json
{
  "mcpServers": {
    "perplexity-ask": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "PERPLEXITY_API_KEY",
        "mcp/perplexity-ask"
      ],
      "env": {
        "PERPLEXITY_API_KEY": "YOUR_API_KEY_HERE"
      }
    }
  }
}
```

### Docker イメージのビルド

※パスは実際にインストールした場所に置き換えてください

```
docker build -t mcp/perplexity-ask:latest -f ~/tools/mcp-server/perplexity-ask/Dockerfile .
```

#### 検索パラメータのカスタマイズ

- 検索パラメータをカスタマイズするには、`index.ts`スクリプト内のAPI呼び出しを直接変更できます
- 詳細な設定オプションは[Perplexity公式APIドキュメント](https://docs.perplexity.ai/api-reference/chat-completions)を参照してください
- 検索結果の言語や地域、検索フィルターなど様々なパラメータを調整可能です
- 主な設定パラメータ例：
  - `model`: 使用するモデル（例：`sonar-deep-research`、`sonar-reasoning-pro`、`sonar-pro`、`sonar`、`r1-1776`）
    - `sonar-deep-research`: 最も高度な検索・分析能力（コンテキスト長: 128k）
    - `sonar-reasoning-pro`: 推論能力に優れ、Chain of Thought（CoT）対応（コンテキスト長: 128k）
    - `sonar-pro`: 高度な検索機能と質問フォローアップ対応（コンテキスト長: 200k）
    - `sonar`: 軽量で速い検索モデル（コンテキスト長: 128k）
    - `r1-1776`: 検索機能を使わないオフラインチャットモデル（コンテキスト長: 128k）
  - `messages`: 会話履歴の配列（各メッセージは`role`と`content`を含む）
  - `temperature`: 応答のランダム性（0.0〜1.0）
  - `max_tokens`: 生成する最大トークン数
  - `top_p`: 生成するトップPの値
  - `stream`: ストリーミングモードの有効/無効（true/false）

#### 検索パラメータの例

```typescript:index.ts
{
  "model": "sonar-pro",
  "messages": [
    {
      "role": "system",
      "content": "あなたは役立つアシスタントです。最新の情報を提供してください。"
    },
    {
      "role": "user",
      "content": "東京の現在の天気はどうですか？"
    }
  ],
  "temperature": 0.7,
  "max_tokens": 2000,
  "top_p": 1,
  "stream": false
}
```

#### トラブルシューティング

問題が発生した場合は以下の方法で解決できます：

- Claudeのトラブルシューティングガイドを参照
- `api@perplexity.ai`にサポートを依頼
- [GitHub Issues](https://github.com/ppl-ai/api-discussion/issues)でバグを報告

---

## 📝 まとめ

Perplexity MCP Serverは、Perplexity の Sonar API をAI との会話から呼び出すようにするコネクタです。このツールを使えば、Web検索をAIの会話に自然に統合し、最新情報に基づいた正確な回答を得られます。Reasoningモデルによる高度な推論、Deep Researchによる複数情報源からの総合的な調査、そして情報源の明示による信頼性の向上など、Perplexityならではの強力な機能を活用できます。複雑な情報収集からレポート作成までのワークフローが効率化され、常に最新のWeb情報にアクセスしながら、AIのハルシネーション（事実に基づかない誤った情報生成）を減少させることが可能です。

ぜひ Perplexity MCP Serverを導入して、AIとのウェブ情報活用を効率化してみてください！

## 📚 参考リンク

- [ppl-ai/modelcontextprotocol - Github](https://github.com/ppl-ai/modelcontextprotocol)
