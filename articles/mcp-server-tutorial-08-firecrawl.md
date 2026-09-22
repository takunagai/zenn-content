---
title: "【MCPのトリセツ #8】Firecrawl MCP：スクレイピングでウェブ情報を取得・分析"
emoji: "🐸"
type: "tech"
topics: ["mcp", "firecrawl", "claude", "claudecode", "codex"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズです。今回は Firecrawl MCP を取り上げます。JavaScript で描画されるページの取得、サイト全体のクロール、検索、構造化データの抽出を AI から実行できます。

:::message
**更新日: 2026-09-23**（初版: 2025-03-08）

料金プラン、ツール構成、導入方法を現行の内容に更新しています。旧版で触れていた Deep Research 機能は廃止され、検索と Agent 機能に置き換わりました。

2026-09-23: 新機能「Alexandria」（データ API のカタログ）への対応と、それに伴う検索の既定の変更を追記しました。
:::

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. [Slack MCPサーバー：チームコミュニケーションを強化](./mcp-server-tutorial-07-slack)
8. **Firecrawl MCP：スクレイピングでウェブ情報を取得・分析（この記事）**
9. [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)
11. [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
12. [Blender MCP Server: 会話で Blender を操作し3Dモデルを作成](./mcp-server-tutorial-12-blender)
13. [Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行](./mcp-server-tutorial-13-perplexity)
14. [国土交通省がMCPサーバーを公開：AI時代のオープンデータ活用45選](./mcp-server-tutorial-14-milt-data)

資料: [ウェブ情報を取得するMCPの比較 (Fetch、Firecrawl、Markdownify、Perplexity)](./mcp-server-tutorial-reference-web-mcp)

---

## Firecrawl MCP でできること

Claude や Codex には標準の Web 検索・Web 取得機能があります。それでも Firecrawl を足す理由は、標準機能では届かない取得ができるからです。

| 機能 | ツール名 | 内容 |
|---|---|---|
| スクレイピング | `firecrawl_scrape` | 1 ページを Markdown や JSON で取得。JavaScript で描画されるページにも対応。Alexandria のツールの実行もこれで行う |
| サイトマップ取得 | `firecrawl_map` | サイト内の URL を一覧にする |
| クロール | `firecrawl_crawl` | 起点の URL から複数ページをたどって取得する |
| 検索 | `firecrawl_search` | Web 検索し、結果のページ本文まで取得する。Alexandria のツールも一緒に探す |
| データ API の検索 | `firecrawl_find_tools` | Alexandria のカタログから、ツールの入力と価格を確かめる |
| ブラウザ操作 | `firecrawl_interact` | クリックや入力をしてから内容を取得する |
| Agent | `firecrawl_agent` | 目的を伝えると、検索と取得を自律的に繰り返して調べる |
| ファイル解析 | `firecrawl_parse` | PDF などのファイルを解析する |

このほかに、論文検索（`firecrawl_research_*`）、ページの変更監視（`firecrawl_monitor_*`）、クレジット残量の確認（`firecrawl_credit_usage`）があります。

> Fetch、Firecrawl、Markdownify、Perplexity の使い分けは、[ウェブ情報を取得するMCPの比較](./mcp-server-tutorial-reference-web-mcp)にまとめています。

## 料金プラン

Firecrawl は無料枠のある有料サービスです。2026 年 9 月 22 日時点の[公式料金ページ](https://www.firecrawl.dev/pricing)の内容は次のとおりです。

| プラン | 月額（月払い / 年払い） | クレジット / 月 | 同時リクエスト数 |
|---|---|---|---|
| Free | $0 | 1,000 | 2 |
| Hobby | $19 / $16 | 5,000 | 5 |
| Standard | $99 / $83 | 100,000 | 25 |
| Growth | $399 / $333 | 500,000 | 50 |

クレジットの消費は、スクレイピング・クロール・マップが 1 ページあたり 1、検索が 10 件あたり 2、ブラウザ操作が 1 分あたり 2 です。Agent は 1 日 5 回まで無料で、それ以降は内容に応じた変動課金になります。Free プランのクレジットは毎月補充されるので、個人の調べ物なら無料枠で足りる場面が多いはずです。

## セットアップ手順

### API キーの取得

[Firecrawl 公式サイト](https://www.firecrawl.dev/)でアカウントを作成し、ダッシュボードで API キー（`fc-` で始まる文字列）を発行します。

> Free プランでも毎月 1,000 クレジットを使えます（クレジットカードの登録は不要）。これから登録する方は、[こちらの紹介リンク](https://www.firecrawl.dev/referral?rid=W385F95R)から登録してもらえると、この連載を続ける励みになります。

### Claude Code

```bash
claude mcp add firecrawl -e FIRECRAWL_API_KEY=fc-YOUR_API_KEY -- npx -y firecrawl-mcp
```

全プロジェクトで使う場合は `-s user` を付けます。

### Codex

```bash
codex mcp add firecrawl --env FIRECRAWL_API_KEY=fc-YOUR_API_KEY -- npx -y firecrawl-mcp
```

### Claude Desktop

Firecrawl はリモート MCP サーバーも提供しています。「Customize > Connectors」で「+」を押し、「Add custom connector」に次の URL を登録すると、OAuth で Firecrawl アカウントに接続できます。API キーを設定ファイルに書かずに済む方法です。

```text
https://mcp.firecrawl.dev/v2/mcp-oauth
```

ローカルで動かす場合は、設定ファイル（開き方は[シリーズ #1](./mcp-server-tutorial-01-install) を参照）に次を追加します。

```json
{
  "mcpServers": {
    "mcp-server-firecrawl": {
      "command": "npx",
      "args": ["-y", "firecrawl-mcp"],
      "env": {
        "FIRECRAWL_API_KEY": "YOUR_API_KEY_HERE"
      }
    }
  }
}
```

旧版の記事では再試行回数やクレジット警告しきい値の環境変数を設定していましたが、現行の README には記載がなくなっているため外しました。

### CLI とスキルという選択肢

Firecrawl は 2026 年 1 月に、公式 CLI とエージェント向けスキルを公開しました。

```bash
npx -y firecrawl-cli@latest init --all --browser
```

Firecrawl MCP は 20 個を超えるツールがあり、接続するとその定義がコンテキストに載ります。CLI + スキルの方式は、常時読まれるのがスキルの名前と説明だけで、実際の取得はコマンドとして実行されます。取得結果がファイルに保存されるので、大きなページを読んでも会話のコンテキストを圧迫しにくい利点もあります。

私は Claude Code ではこの CLI + スキルを使っています。コマンドを実行できないチャット中心の Claude Desktop では、MCP の方が向きます。MCP とスキルの選び方は[シリーズ #1](./mcp-server-tutorial-01-install) にまとめました。

## 基本的な使い方（プロンプト）

```text
このページの内容を取得して要点をまとめて
https://example.com
```

```text
https://example.com のサイト構造を調べて、ドキュメントに当たるページの URL を一覧にして
```

```text
「クラウドネイティブアプリケーション開発」の最新情報を検索して、主要なトレンドと技術をまとめて
```

```text
この EC サイトから、製品名、価格、説明を抽出して表にまとめて
https://example-shop.com/products
```

他の Web 取得ツールが先に呼ばれてしまうときは、「Firecrawl で」と添えると確実です。

## Alexandria（データ API のカタログ）を使う

Firecrawl は 2026 年 9 月 22 日（米国時間）に「Alexandria」を公開しました。公式データプロバイダーの API や Firecrawl 独自のインデックス（論文・開発者向けドキュメントなど）を、Firecrawl から探して実行できるカタログです。npm のダウンロード数、GitHub の issue、米国株の株価、SEC の開示書類など、Web ページを読むより API から取った方が正確なデータを、エージェントが自分で見つけて取りに行けます。

MCP では、`firecrawl_search` でツールを探し、`firecrawl_find_tools` で入力と価格を確かめ、`firecrawl_scrape` で実行します。**ツールを探すのは無料で、実行したときだけ、ツールごとに決まったクレジットを消費します。**

:::message alert
**`firecrawl_search` の既定が変わっています**

API キーまたは OAuth（上のリモート版）で認証している場合、`firecrawl_search` の既定の検索先は **Web と Alexandria の両方**になりました（`firecrawl-mcp` 3.25.2 で確認）。両方の検索は同時に走るので、ツールが見つかっても Web 検索の分は従来どおり課金されます。ツールの検索自体は無料なので、料金が増えるわけではありません。

Web の検索結果だけが欲しいときは、「sources は web だけで検索して」と指示してください。
:::

頼むときは、実行する前に価格を確認させるのが安全です。

```text
Firecrawl の Alexandria で、npm の hono と express の過去 1 か月のダウンロード数を調べて。
実行する前に、使うツールと 1 回あたりのクレジットを見せて、私の了承を取ってから実行して。
```

```text
Alexandria で、SEC に提出された Apple の最新の 10-K から「Risk Factors」の節を取り出して、主なリスクを 5 つに要約して。
合計 10 クレジットを超えそうなら、その時点で止めて相談して。
```

実際に検索してみた範囲では、米国のサービスのツールが多く出てきました（2026-09-23 時点）。日本の情報は見つからないことが多いので、まずは無料の探索の段階で対象地域を確かめさせてください。

## 活用テクニック

### 競合サイトの比較レポート

```text
https://competitor1.com
https://competitor2.com
https://competitor3.com
これらのサイトを取得して、主要な機能、料金体系、コンテンツの構成、想定しているユーザー層を比較表にして
```

### ドキュメントサイトをまとめて取得する

```text
https://docs.example.com/guide/ 以下をクロールして。上限は 30 ページ、サブドメインは含めない。各ページのタイトルと要約を一覧にして
```

クロールはページ数の分だけクレジットを消費します。上限ページ数を必ず指定し、先に `firecrawl_map` で URL の数を確認してから実行するのが安全です。

### 操作が必要なページを取得する

```text
https://example.com/pricing を開いて、「年払い」のタブに切り替えてから料金表を取得して
```

### Agent に調査を任せる

```text
Firecrawl の Agent で、国内の主要な会計 SaaS 5 社の料金プランを調べて、プラン名・月額・主な機能を表にまとめて。出典の URL も付けて
```

旧版の記事にあった「ディープリサーチ」のプロンプトは、この Agent 機能が後継にあたります。

## 他の MCP サーバーとの組み合わせ

[Filesystem MCP](./mcp-server-tutorial-02-filesystem) と組み合わせると、取得した内容をそのまま手元に保存できます。

```text
https://documentation.example.com のチュートリアルのページを取得して、Markdown に変換し、~/Documents/tutorials/ に 1 ページ 1 ファイルで保存して
```

## 使用上の注意点

- **クレジット消費**: クロールと Agent は消費が読みにくい機能です。上限を指定し、`firecrawl_credit_usage` で残量を確認しながら使います
- **利用規約と robots.txt**: 取得先サイトの利用規約に従います。短時間に大量のリクエストを送ると、サイト側に負荷をかけたりブロックされたりします
- **取得した文章は外部の入力**: Web ページには AI への指示を装った文章が紛れていることがあります。取得結果をもとに AI がファイル操作や送信をしようとしたら、内容を確認してから許可します
- **抽出データの検証**: 構造化抽出の結果は、元のページと突き合わせて確認します
- **Alexandria の実行料金**: ツールごとに消費クレジットが違います。実行前に価格を確認させます。また、一部のプロバイダーはデータ利用規約への同意を求めます。同意はエージェントに任せず、人が内容を確認してから行います

## まとめ

- Firecrawl MCP は、標準の Web 取得では届かない JavaScript 描画ページ、サイト全体のクロール、操作を伴う取得を担います
- 導入は `npx -y firecrawl-mcp` か、OAuth で接続するリモート版です。Claude Code や Codex では CLI + スキルも選択肢になります
- Free プランは月 1,000 クレジットです。クロールと Agent は上限を決めて使います
- Alexandria で、Web ページではなくデータ API から正確な値を取れるようになりました。探すのは無料、実行はツールごとの料金です

新しい MCP 記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。

## 参考リンク

- [Firecrawl 公式サイト](https://www.firecrawl.dev/)
- [firecrawl/firecrawl-mcp-server - GitHub](https://github.com/firecrawl/firecrawl-mcp-server)
- [Firecrawl ドキュメント](https://docs.firecrawl.dev/)
- [Introducing Firecrawl Skill and CLI - Firecrawl Blog](https://www.firecrawl.dev/blog/introducing-firecrawl-skill-and-cli)
- [Introducing Alexandria and our $75M Series B - Firecrawl Blog](https://www.firecrawl.dev/blog/introducing-alexandria-series-b)
- [Alexandria - Firecrawl ドキュメント](https://docs.firecrawl.dev/features/alexandria)

次回は、さまざまなファイルを Markdown 化できる「[Markdownify MCP Server](./mcp-server-tutorial-09-markdownfy)」を解説します。

## さいごにおねだり

この記事が役に立ったら、[こちらの紹介リンク](https://www.firecrawl.dev/referral?rid=W385F95R)から Firecrawl に登録してもらえるとうれしいです。Free プランでも毎月 1,000 クレジットを使えるので、まずは無料の範囲で試せます（[料金ページ](https://www.firecrawl.dev/pricing)、2026 年 9 月 22 日時点）。
