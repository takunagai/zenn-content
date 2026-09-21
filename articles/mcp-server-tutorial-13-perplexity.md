---
title: "【MCPのトリセツ #13】Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行"
emoji: "🐸"
type: "tech"
topics: ["mcp", "perplexity", "claude", "claudecode", "codex"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズです。今回は Perplexity 公式の MCP サーバーを取り上げます。Perplexity の検索と、検索結果を踏まえた回答・調査・推論を、AI との会話から呼び出せます。

:::message
**更新日: 2026-09-22**（初版: 2025-03-16）

旧版は、リポジトリをクローンして Sonar API のキーで動かす手順でした。現在は npm パッケージ `@perplexity-ai/mcp-server` が公開され、1 行で導入できます。また、Perplexity は「Sonar Chat Completions は Agent API に移行し、Sonar のサポートは 2026 年 9 月 27 日まで」と告知しています。公式 MCP サーバーはすでに Agent API ベースに切り替わっているので、旧版の手順で導入した方は入れ直しをおすすめします。
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
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)
11. [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
12. [Blender MCP Server: 会話で Blender を操作し3Dモデルを作成](./mcp-server-tutorial-12-blender)
13. **Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行（この記事）**
14. [国土交通省がMCPサーバーを公開：AI時代のオープンデータ活用45選](./mcp-server-tutorial-14-milt-data)

資料: [ウェブ情報を取得するMCPの比較 (Fetch、Firecrawl、Markdownify、Perplexity)](./mcp-server-tutorial-reference-web-mcp)

---

## Perplexity MCP Server でできること

[perplexityai/modelcontextprotocol](https://github.com/perplexityai/modelcontextprotocol) は Perplexity 公式の MCP サーバーです。旧版の時点ではツールが `perplexity_ask` の 1 つだけでしたが、現在は用途別の 4 つに整理されています。

| ツール | 用途 | 裏側の仕組み |
|---|---|---|
| `perplexity_search` | Web 検索。順位付きの検索結果をそのまま返す | Search API |
| `perplexity_ask` | 検索を踏まえた一般的な質問への回答 | Agent API の `fast` プリセット |
| `perplexity_reason` | 比較検討や段階的な推論が要る質問 | Agent API の `medium` プリセット |
| `perplexity_research` | 複数の情報源を読み込む深い調査 | Agent API の `high` プリセット |

旧版のサーバーは `sonar-pro`・`sonar-reasoning-pro`・`sonar-deep-research` といったモデルを直接指定していました。現行版ではこれらの指定はツールの定義から外れ、送っても無視されます。

Claude や Codex には標準の Web 検索があります。それでも Perplexity を足す場面は、出典付きの調査レポートをまとめて取りたいときと、検索結果の一覧だけを軽く取りたいときです。前者は `perplexity_research`、後者は `perplexity_search` が向いています。

## 料金

Perplexity の API は従量課金で、Perplexity の Pro プラン（月額のサブスクリプション）とは別に支払いが発生します。2026 年 9 月 22 日時点の[公式料金ページ](https://docs.perplexity.ai/docs/getting-started/pricing)の内容は次のとおりです。

| API | 料金 |
|---|---|
| Search API | 1,000 リクエストあたり $5 |
| Agent API | 使用するモデルのトークン料金 + ツール呼び出し料金（`web_search` は 1 回 $0.0025、`fetch_url` は 1 回 $0.0005） |

`perplexity_research` は 1 回の調査で多数の検索とページ取得を行うため、他のツールより費用がかかります。最初は `perplexity_search` や `perplexity_ask` から試し、API コンソールで利用額を確認しながら使うのが安全です。

旧版では「Pro プランには毎月 5 ドル分の API クレジットが付く」と書いていましたが、現在の公式料金ページにはその記載が見当たりません。Pro プランの方は、API の管理画面でクレジットの有無を確認してください。

## セットアップ手順

### API キーの取得

1. [Perplexity の API コンソール](https://console.perplexity.ai/project/keys)を開く
2. 支払い方法を登録する（API は従量課金で、サブスクリプションは不要）
3. 「API Keys」のページでキーを発行して控える

### Claude Code

```bash
claude mcp add perplexity --env PERPLEXITY_API_KEY="your_key_here" -- npx -y @perplexity-ai/mcp-server
```

### Codex

```bash
codex mcp add perplexity --env PERPLEXITY_API_KEY="your_key_here" -- npx -y @perplexity-ai/mcp-server
```

### Claude Desktop

設定ファイル（開き方は[シリーズ #1](./mcp-server-tutorial-01-install) を参照）に次を追加し、Claude Desktop を再起動します。

```json
{
  "mcpServers": {
    "perplexity": {
      "command": "npx",
      "args": ["-y", "@perplexity-ai/mcp-server"],
      "env": {
        "PERPLEXITY_API_KEY": "your_key_here"
      }
    }
  }
}
```

### 環境変数

| 変数 | 内容 |
|---|---|
| `PERPLEXITY_API_KEY` | API キー（必須） |
| `PERPLEXITY_TIMEOUT_MS` | タイムアウト。既定は 5 分。`perplexity_research` が途中で切れるときに延ばす |
| `PERPLEXITY_LOG_LEVEL` | ログの詳細度（`DEBUG`・`INFO`・`WARN`・`ERROR`）。既定は `ERROR` |

## プロンプトのサンプル

標準の Web 検索や他の MCP が先に呼ばれることがあるので、「Perplexity で」と添えると確実です。調査の深さを指定したいときは「Perplexity の research で」のようにツールまで伝えます。

### 基本的な検索と質問

```text
Perplexity で、日本の最新の経済指標を調べて、現在の経済状況を分析して
```

```text
Perplexity で、React と Vue.js の最新バージョンの違いを比較し、それぞれの長所と短所をまとめて
```

```text
Perplexity の search で、「MCP Registry」に関する直近 1 か月の記事を探して、タイトルと URL を一覧にして
```

### 推論が要る比較検討（perplexity_reason 向け）

```text
Perplexity の reason で、次の手順で [製品名] と競合製品を比較して。

1. [製品名] の主要な競合製品を 3〜5 つ特定する
2. 各製品の価格体系、主要機能、ユーザーレビューの傾向を集める
3. 各製品の強みと弱みを分析する
4. すべての情報源を引用して、比較表にまとめる
```

### 深い調査（perplexity_research 向け）

```text
Perplexity の research で、[技術分野] の今後 3 年間のトレンドを調査して。

1. 最新の研究論文、業界レポート、専門家の意見から、主要な技術的進歩を特定する
2. それらが利用者の行動、ビジネスモデル、規制に与える影響を分析する
3. 今後 3 年間の予測タイムラインを作り、重要なマイルストーンを示す
4. すべての情報源を明記する
```

```text
Perplexity の research で、[研究テーマ] に関する過去 3 年間の重要な学術論文を 8〜10 件特定し、各論文の研究目的、方法、主な結論を要約して。論文間で一致している点と食い違っている点も整理して
```

調査の範囲、期間、件数、出力の形を先に指定しておくと、結果のばらつきが小さくなります。`[ ]` の部分は具体的な内容に置き換えてください。

## 使用上の注意点

- **費用は使った分だけ増える**: AI が自律的に何度もツールを呼ぶ場面では、想定より費用が伸びます。ツールの実行を自動承認にする前に、1 回あたりの費用感を確かめておきます
- **出典を確認する**: Perplexity は出典を付けて回答しますが、要約の過程で内容がずれることはあります。意思決定に使う情報は出典のページを開いて確かめます
- **API キーの扱い**: 設定ファイルに書いたキーは平文で保存されます。チャット欄には貼らず、漏れた可能性があるときは管理画面で失効させます

## まとめ

- 導入は `npx -y @perplexity-ai/mcp-server` の 1 行になりました。リポジトリのクローンとビルドは不要です
- ツールは search・ask・reason・research の 4 つです。Sonar のモデル指定は廃止され、Agent API のプリセットに置き換わりました
- 料金は Pro プランとは別の従量課金です。research は費用がかかるので、用途を決めて使います

## 参考リンク

- [perplexityai/modelcontextprotocol - GitHub](https://github.com/perplexityai/modelcontextprotocol)
- [Perplexity MCP Server - Perplexity Docs](https://docs.perplexity.ai/docs/getting-started/integrations/mcp-server)
- [Pricing - Perplexity Docs](https://docs.perplexity.ai/docs/getting-started/pricing)

次回は「[国土交通省の MCP サーバー](./mcp-server-tutorial-14-milt-data)」を取り上げます。Web 情報を取得する MCP の使い分けは、[比較記事](./mcp-server-tutorial-reference-web-mcp)にまとめています。
