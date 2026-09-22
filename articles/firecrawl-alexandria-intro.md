---
title: "Firecrawl の新機能 Alexandria 入門 ─ AI エージェントに 500 超のデータ API を「探して・読んで・叩かせる」"
emoji: "🐸"
type: "tech"
topics: ["firecrawl", "ai", "ai駆動開発", "スクレイピング", "mcp"]
published: true
---

![古代図書館を思わせる柱廊を背に、木製のカード目録から 1 枚のカードを抜き出して眺める眼鏡の女性のイラスト](/images/firecrawl-alexandria-intro/eyecatch.webp)

Web スクレイピング API の [Firecrawl](https://www.firecrawl.dev/referral?rid=W385F95R) が、2026 年9月22日（米国時間）に新機能「Alexandria」を公開しました。AI エージェントが、公式データプロバイダーの API・Firecrawl 独自のインデックス・ライブの Web を、同じ Firecrawl API から使えるようにする仕組みです。

一言でいえば **「データ API のカタログと実行窓口を 1 つにまとめたもの」** です。エージェントは自然文でツールを探し、入力と価格を読み、そのまま実行できます。探すところまでは無料で、お金がかかるのは実行したときだけです。

この記事では、機能と料金を整理したうえで、Claude Code などの AI エージェントに使わせるときの指示のサンプルや、Node.js の SDK での基本サンプルコードを、再現できる形で紹介します。

:::message
**対象読者**: Firecrawl を使ったことがある、または AI エージェントに外部データを取らせたいエンジニア

**検証環境**（2026-09-23 時点）
- Node.js 22 以上（SDK の要件。検証は v26.5.1）
- SDK: `firecrawl` 4.41.0（npm）
- CLI: `firecrawl-cli` 1.24.3
:::

:::message alert
**CLI を使っている人へ: `firecrawl search` の既定が変わっています**

CLI の `firecrawl search` は、1.19.27 では既定の検索先が Web だけでしたが、1.24.3 では **Web と Alexandria の両方**になっています。Web 検索とツール検索は同時に走るので、ツールが見つかっても Web 検索の分は従来どおり課金されます（10 件ごとに 2 クレジット）。

目的に合わせて、次の 3 つの書き方を使い分けてください。

| やりたいこと | 書き方 | 課金 |
|---|---|---|
| ツールだけを探す | `firecrawl search alexandria "..."` | 無料 |
| Web だけを検索する（従来の動き） | `firecrawl search "..." --sources web` | Web 検索分 |
| Web とツールを両方見る（新しい既定） | `firecrawl search "..."` | Web 検索分 |

Web 検索だけのつもりで使っているスクリプトやエージェントの設定があるなら、`--sources web` を付けておくと以前と同じ結果の形に戻せます。詳しくは後半の「ハマりどころ」で説明します。
:::

## Alexandria とは

Alexandria という名前は、古代のアレクサンドリア図書館から取られています。公式ブログの言葉を借りると「超知能のための図書館」です。

収録されているデータ源は、プレスリリースによると次のとおりです。

| 種類               | 中身                                           |
| ---------------- | -------------------------------------------- |
| 公式データプロバイダー      | 有償契約で接続している提供元の API（Wikimedia Enterprise など） |
| Research Index   | 数千万件の学術論文アブストラクト                             |
| Developer Index  | ドキュメント・README・Issue・マージ済み PR など数千万件          |
| Government Index | 法律・規則・条例                                     |
| サイト専用コネクタ        | 特定サイトの情報やデータセットを直接取るためのもの                    |
| ライブの Web         | 従来の search / scrape                          |

発表時点のプロバイダー網は、88 のプロバイダーが 28 カテゴリで 504 の機能（capability）を提供し、インデックス済みのソースは 1 億 1,300 万件超とされています[^pr]。

発表は、Smash Capital が主導する 7,500 万ドルのシリーズ B 調達と同時でした。調達資金の使い道として、知識を提供する人や組織に、AI エージェントが使った分だけ報酬を払う仕組みを挙げています。個人やクリエイターも参加できるセルフサービス型の窓口を「近く開く」予定とのことです。

:::message
2026-09-23 時点で収録されているデータは米国のサービスが中心です。求人・商品・株価などは、日本の情報を期待すると空振りが多いです。
:::

## 何が楽になるのか

例えば、AI エージェントに企業を調べさせる場面を考えてみます。会社のサイトは読めても、財務データや開示書類は別の API にあります。それぞれに独自のキー、料金体系、レスポンス形式がある。組み込む手間を惜しんでデータ源を 1 つ外すと、今度は回答の質が落ちます。どれほど強いモデルでも、見つけていない情報からは推論できません。

Alexandria はこの「API を 1 つずつ組み込む」作業を肩代わりします。エージェント側が覚えるのは Firecrawl の呼び方だけです。どのプロバイダーにどんな機能があり、入力に何が要り、1 回いくらかかるのかは、カタログに問い合わせれば返ってきます。

公式ブログやドキュメントに出てくる使い道は、たとえば次のようなものです。

- **競合調査**: スタートアップのディレクトリを横断して、同じ領域の企業と実際の事業内容を比べる
- **見込み客探し**: 条件に合う企業を探し、担当者の情報まで補完する
- **論文の比較**: Research Index から関連研究を引く
- **技術調査**: Developer Index から一次情報（Issue・PR・README）を引く。パッケージのダウンロード数のような統計も取れる

Firecrawl は、社内評価で Alexandria を使ったエージェントの回答品質が、組み込みの Web ツールを使った場合より 21% 高かったと発表しています（同じモデルとプロンプトで 845 タスク、AI によるブラインド採点）[^blog]。ただしこれは提供元の自社評価です。

## 料金

Alexandria 自体に別料金のプランはありません。通常の Firecrawl のクレジットを消費します。

| 操作 | 料金 |
|---|---|
| ツールを探す（`sources: ["alexandria"]` の検索、`findTools`） | 無料 |
| ツールを実行する | ツールごとに提示されるクレジット |
| Web 検索を併用する（`sources: ["web", "alexandria"]`） | Web 検索分は通常どおり課金（10 件ごとに 2 クレジット） |

実行料金はツールごとに違い、カタログの `creditsCost` に書かれています。今回試した npm のダウンロード数を取るツールは 1 回 1 クレジットでした。実行結果にも `creditsCost` が返るので、実際に何クレジット使ったかはレスポンスで確認できます。

クレジットの元になるプランは次のとおりです（年払い時の月額。2026 年 9 月 4 日改定の料金表[^pricing]）。

| プラン | 月額 | クレジット / 月 |
|---|---|---|
| Free | $0 | 1,000 |
| Hobby | $16（月払いは $19） | 5,000 |
| Standard | $83 | 100,000 |
| Growth | $333 | 500,000 |
| Scale | $599 | 1,000,000 |

Free プランでもクレジットカードなしで毎月 1,000 クレジットが付くので、この記事の手順を試す程度なら無料枠で足ります。

## AI エージェントから使う（MCP / CLI）

Claude Code などのエージェントに使わせるなら、コードを書く必要はありません。公式ドキュメントによると、MCP で接続したエージェントは次の 3 つのツールを使い分けます。

| MCP ツール | 役割 |
|---|---|
| `firecrawl_search` | ツールを探す |
| `firecrawl_find_tools` | ツールの入力と価格を確かめる |
| `firecrawl_scrape` | ツールを実行する |

MCP サーバーの設定方法は[Firecrawl 公式ドキュメント](https://docs.firecrawl.dev/mcp-server)にまとまっています。
Firecrawl MCP の導入から使い方までは、以前に書いたこちらの記事でも紹介しています。
https://zenn.dev/takna/articles/mcp-server-tutorial-08-firecrawl

CLI とエージェント用スキルを入れる場合、公式ブログは次のコマンドを案内しています。実行後にエージェントを再起動すると、スキルが読み込まれます（この手順は未検証です）。

```bash
npx -y firecrawl-cli@latest init --all --browser
```

### 実用的なプロンプト例

エージェントへの頼み方で効くのは、**「実行する前に入力と価格を確認させる」**ことと、**「使ってよいクレジットの上限を決める」**ことの 2 つです。探索は無料なので、そこで契約を読ませてから有料の実行に進ませると、想定外の出費を防げます。

どの用途にも使える型は、次のようなものです。

```text
Alexandria で「（知りたいこと）」に使えるツールを探して。
実行する前に、使うツール・入力・1回あたりのクレジットを一覧で見せて、私の了承を取ってから実行して。
合計10クレジットを超えそうなら、その時点で止めて相談して。
取得したデータには、取得元の URL を必ず添えて。
```

以下は、この型を用途別に具体化した例です。どれも 2026-09-23 時点で、無料のツール検索で対応するツールが返ってくることを確かめています（実行して結果を確かめたのは npm のダウンロード数だけです）。

:::message
2026-09-23 時点では、日本の情報を期待すると空振りしやすいので、まずは探索の段階で対象地域を確かめさせてください。
:::

#### 開発・技術選定

ライブラリの比較は、ダウンロード数・バージョン履歴・依存されている数を 1 回の依頼で集められます。

```text
npm の hono・express・fastify を比較したい。
Alexandria で、過去 1 か月のダウンロード数、最新バージョンと公開日、依存しているパッケージ数を取得して表にまとめて。
実行前に使うツールと合計クレジットを見せて。
```

GitHub のリポジトリの状態を確かめるときにも使えます。

```text
導入を検討している GitHub リポジトリ honojs/hono について、
Alexandria のツールで、open と closed の issue 数と、直近に作られた issue を10件取得して。
メンテナンスが続いているかを判断する材料として要約して。
```

```text
Hugging Face で、日本語に対応したテキスト埋め込みモデルを探して。
ダウンロード数の多い順に5件、ライセンスと過去30日のダウンロード数を表にして。
商用利用できないライセンスのものには印を付けて。
```

#### 論文・一次情報のリサーチ

Research Index は、論文の検索・関連論文・本文の該当箇所の抜き出しを分けて呼べます。

```text
「RAG の検索精度を上げる手法」についての論文を Alexandria の Research Index で探して。
上位5本のタイトル・発表年・要点を一覧にして、いちばん関連が強い1本については、それを引用している論文も3本挙げて。
```

#### 競合調査・営業リスト

公式ブログの事例にもある使い方です。Y Combinator の企業プロフィールや、企業の役職者を引くツールがあります。

```text
AI のカスタマーサポートツールを作っている Y Combinator 出身のスタートアップを調べて。
各社のバッチ、所在地、従業員規模、事業内容を表にして、事業内容が似ている順に並べて。
```

```text
（企業名）で、マーケティングとパートナーシップを担当している役職者を調べて。
名前と役職だけでよい。連絡先の取得は、私が許可するまで実行しないで。
```

:::message alert
業務用メールアドレスや携帯電話番号を取るツールもあります。携帯電話番号のツールは、カタログの説明で最も高価な部類とされています。どちらも個人情報なので、取得の可否と用途は人が判断してください。
:::

#### 金融・市場データ（米国）

```text
米国株の NVDA と AMD について、現在の株価と主要な指標を取得して。
あわせて過去3か月の日次終値を取って、値動きを比べて。
```

```text
SEC に提出された Apple の最新の 10-K から、「Risk Factors」の節だけを取り出して、主なリスクを 5 つに要約して。
```

```text
Nasdaq のニュースから、TSLA に関する最新の記事の見出しを10件集めて、公開日時と一緒に一覧にして。
```

#### 店舗・商品・求人

```text
Google マップ上の（店舗名・住所）のクチコミを取得して、評価の低いクチコミに共通する不満を3つにまとめて。
```

```text
米国の求人サイトで「AI engineer」の求人をサンフランシスコで探して、募集している企業と職種を一覧にして。
```

```text
Amazon.com の（商品名）について、表示されている評価と件数を取得して、競合の（商品名）と比べて。
```

#### ポッドキャスト

```text
AI エージェントについて話しているポッドキャストのエピソードを探して、該当する箇所の文字起こしだけを読んで、主な論点を要約して。
エピソード全体の文字起こしは高くつくので、まずは該当区間だけにして。
```

ポッドキャストの文字起こしのツールは、エピソード全体・プレビュー・区間の 3 種類があり、カタログの説明によると区間の方が安く済みます。このように、同じ目的でも粒度の違うツールが並んでいることがあるので、「安い方から試して」と一言添えると無駄が減ります。

## Node.js で使う 3 ステップ

### 0. 準備

Firecrawl の API キーを持っていて、SDK も最新版（この記事では 4.41.0）を導入済みなら、この節は飛ばして「1. 探す」へ進んでください。

Firecrawl のアカウントを作り、ダッシュボードで API キーを発行します。アカウントは次のリンクから作れます。Free プランならクレジットカードの登録は要りません。
[» Firecrawl ホームページ](https://www.firecrawl.dev/referral?rid=W385F95R)

作業用のディレクトリで SDK を入れます。Alexandria 対応の `findTools` などは新しめのバージョンで入ったので、古い SDK を使っている場合は更新してください。

```bash
npm init -y
npm install firecrawl
```

API キーは `.env` に置き、Node.js の `--env-file` で読み込みます。
**※ コードに直接書かないでください。**

```bash:.env
FIRECRAWL_API_KEY=fc-xxxxxxxxxxxxxxxx
```

以降のスクリプトは `node --env-file=.env <ファイル名>` で実行します。

### 1. 探す

やりたいことを自然文で書いて検索します。`sources` に `"alexandria"` だけを指定すると、Web 検索はせずツールだけを返します。この場合は無料です。

```js:1-search.mjs
import { Firecrawl } from "firecrawl";

const firecrawl = new Firecrawl({ apiKey: process.env.FIRECRAWL_API_KEY });

const result = await firecrawl.search("npm パッケージの週間ダウンロード数", {
  sources: ["alexandria"],
  limit: 3,
});

for (const tool of result.tools ?? []) {
  console.log(`${tool.provider}/${tool.capability}`);
  console.log(`  ${tool.description}`);
}
```

結果の `tools` に、プロバイダー名（`provider`）と機能名（`capability`）の組が並びます。この組がツールの住所で、実行するときもこの 2 つで指定します。

検索の文は日本語でも通ります（※ 2026-09-23 時点で、日本のデータはまだ少ないです。）。英語の「npm package weekly downloads」で検索したときと同じツールが最上位に来ました。ただし返ってくるツールの説明（`description`）は英語です。
### 2. 入力と価格を確かめる

実行する前に、ツールが何を受け取り、何を返し、いくらかかるのかを確認します。`toolDetail: "full"` を付けると、検索結果にツールの契約（入力と出力の定義）がすべて含まれます。

```js:2-inspect.mjs
import { Firecrawl } from "firecrawl";

const firecrawl = new Firecrawl({ apiKey: process.env.FIRECRAWL_API_KEY });

const result = await firecrawl.search("npm パッケージの週間ダウンロード数", {
  sources: ["alexandria"],
  limit: 3,
  toolDetail: "full",
});

console.log(JSON.stringify(result.tools[0], null, 2));
```

実際の出力を要点だけ抜き出すと、こうなりました。

```json
{
  "provider": "package-registry-metadata-download-stats",
  "capability": "packages/downloads",
  "name": "Daily downloads",
  "creditsCost": 1,
  "options": [
    { "name": "registry", "required": true, "oneOf": ["npm", "pypi", "crates"] },
    { "name": "name", "required": true },
    { "name": "period", "default": "last-month", "oneOf": ["last-week", "last-month", "last-year"] }
  ],
  "response": { "key": "series", "paginated": true }
}
```

`creditsCost: 1` が 1 回の実行料金です。`options` を見れば、`registry` と `name` が必須で、`period` は省略すると `last-month` になることが分かります。エージェントに使わせる場合も、この契約を読ませてから実行させる流れになります。

### 3. 実行する

`scrape` に URL の代わりに `alexandria` を渡すと、ツールを実行します。ページの URL は要りません。

```js:3-run.mjs
import { Firecrawl } from "firecrawl";

const firecrawl = new Firecrawl({ apiKey: process.env.FIRECRAWL_API_KEY });

const response = await firecrawl.scrape({
  alexandria: {
    provider: "package-registry-metadata-download-stats",
    capability: "packages/downloads",
    options: { registry: "npm", name: "firecrawl", period: "last-week" },
  },
});

const [toolResult] = response.alexandria;
if (toolResult.error) {
  console.error(toolResult.error.code, toolResult.error.message);
} else {
  console.log("消費クレジット:", toolResult.creditsCost);
  console.log(toolResult.data);
}
```

結果は `response.alexandria` に配列で入っています（1 回に複数のツールを実行できるため）。公式ドキュメントも、データを使う前にツールごとの `error` を確認するよう書いています。

実際のレスポンスの一部です。

```json
{
  "provider": "package-registry-metadata-download-stats",
  "capability": "packages/downloads",
  "creditsCost": 1,
  "data": {
    "name": "firecrawl",
    "registry": "npm",
    "period": "last-week",
    "start": "2026-09-15",
    "end": "2026-09-21",
    "total": 608951,
    "series": [
      { "date": "2026-09-15", "downloads": 0 },
      { "date": "2026-09-16", "downloads": 150261 }
    ],
    "source_url": "https://api.npmjs.org/downloads/range/last-week/firecrawl"
  },
  "records": 7,
  "upstreamStatus": 200
}
```

`source_url` に取得元の URL が入っているので、値の出どころを後から確かめられます。

### おまけ: カタログを上から潜る

検索ではなく、カタログを分類から順にたどることもできます。`findTools` も無料です。

```js:4-browse.mjs
import { Firecrawl } from "firecrawl";

const firecrawl = new Firecrawl({ apiKey: process.env.FIRECRAWL_API_KEY });

const catalogue = await firecrawl.findTools({ limit: 3 });
console.log(catalogue.level, catalogue.total);

// 各項目の next をそのまま scrape に渡すと、1 階層下が返る
const next = catalogue.items[0]?.next;
if (next) {
  const details = await firecrawl.scrape({ alexandria: next });
  console.log("消費クレジット:", details.creditsCost);
  console.log(JSON.stringify(details.alexandria[0].data, null, 2));
}
```

手元では、最上位が `categories` レベルで `total` は 21 でした（`ai-models`・`apps`・`companies`・`developer`・`finance` など）。`ai-models` の `next` をたどると Hugging Face のプロバイダーが返り、そこからさらに 1 階層下がるとツールの一覧になります。この辿る操作の `creditsCost` は 0 でした。

各項目の `next` は「次に投げるリクエスト」そのものです。エージェントは中身を理解しなくても、`next` を `scrape` に渡し続けるだけでカタログを潜れます。

## ハマりどころ

### 公式ドキュメントの CLI の例が、最新の CLI で通らない

公式ドキュメントの CLI の例にある `find-tools --providers` は、執筆時点の最新版（1.24.3）では次のエラーになりました。

```text
error: unknown option '--providers'
```

1.24.3 で確認できた書き方は次のとおりです。ツールの検索は `search alexandria`、実行は `scrape` に `プロバイダー/機能` を渡します。

```bash
# ツールだけを検索する
npx -y firecrawl-cli@latest search alexandria "npm パッケージのダウンロード数"

# ツールを実行する（1 クレジット）
npx -y firecrawl-cli@latest scrape package-registry-metadata-download-stats/packages/downloads \
  --options '{"registry":"npm","name":"hono","period":"last-week"}'
```

リリース直後でドキュメントと CLI がずれているようです。CLI で動かないときは、`firecrawl <コマンド> --help` で手元のバージョンのオプションを確かめてください。

### Web 検索と併用すると、Web 側は課金される

`sources: ["web", "alexandria"]` にすると、Web ページとツールが一緒に返ってきて便利です。ただし Web 検索の分は通常どおりクレジットを消費します。ツールを探したいだけなら `["alexandria"]` に絞れば無料です。

「まず Alexandria でツールを探し、見つからなければ Web を検索する」という順番ではありません。Web 検索とツール検索は**同時に**実行され、ツールが見つかっても Web 検索の分は課金されます。CLI で確かめた結果は次のとおりです。

| コマンド | 返ってきたもの | 消費クレジット |
|---|---|---|
| `firecrawl search "npm パッケージのダウンロード数" --limit 2` | Web 2 件 + ツール 2 件 | 2 |
| `firecrawl search alexandria "npm パッケージのダウンロード数"` | ツール 4 件 | 0 |
| `firecrawl search "npm パッケージのダウンロード数" --limit 2 --sources web` | Web 2 件 | 2 |

CLI の `search` は、1.24.3 では既定の検索先が `web,alexandria` です（手元にあった 1.19.27 では `web` だけでした）。**ツールだけを探すなら `search alexandria` の形を、ツールが要らないなら `--sources web` を付けてください。** 特に `--sources web` は、既存のスクリプトやエージェントに Web 検索をさせている場合に効きます。付けないと結果に `tools` が加わるので、CLI の出力をそのまま別の処理やエージェントに渡しているなら、一度確かめておくと安心です。

### プロバイダーによっては利用規約への同意が要る

一部のプロバイダーは、実行前にデータ利用規約への同意を求めます。その場合は `THIRD_PARTY_DATA_TERMS_REQUIRED` というエラーが返り、`requiresAction.url` に同意用の URL が入っています。同意できるのは組織の管理者だけで、ドキュメントは、エージェントが規約に同意する前にユーザーの明示的な許可を得る必要があるとしています。今回試したツールではこのエラーは出ていません。

### 返ってきた値は鵜呑みにしない

上の npm のダウンロード数では、7 日間のうち 9 月 15 日と 17 日だけが `0` で、ほかの日は 1 日 7 万〜16 万件でした。取得元の値をそのまま返しているのか、途中で欠けたのかは、このレスポンスだけでは判断できません。数値を記事やレポートに使うときは、`source_url` の取得元と突き合わせてから使うのが安全です。

### 「research」カテゴリは論文インデックスではない

SDK の型定義のコメントに書かれている注意点です。`search()` の `categories: ["research"]` は、arXiv や PubMed などの学術サイトに絞った**普通の Web 検索**です。論文のアブストラクトそのものを検索したいなら、`firecrawl.research.searchPapers()` を使います。名前が似ているので取り違えやすいところです。

## まとめ

- Alexandria は、データ API のカタログと実行窓口を Firecrawl にまとめたもの。公式プロバイダー・独自インデックス・ライブの Web を同じ API で扱える
- 流れは「`search` で探す → `toolDetail: "full"` で入力と価格を読む → `scrape({ alexandria })` で実行する」の 3 ステップ。探すのは無料で、実行したぶんだけ課金される
- リリース直後なので、ドキュメントと CLI のずれがある。収録範囲は英語圏中心。返ってきた値は `source_url` で裏を取る

個別の API を 1 つずつ契約して組み込んでいた作業が、カタログへの問い合わせに置き換わるのが一番の変化です。まずは無料の探索で、自分の用途に合うツールがあるかどうかを確かめるところから始めてみてください。Free プランでも毎月 1,000 クレジットが付くので、ツールの実行も数回なら無料枠で試せます。アカウントをまだ持っていない場合は、次のリンクから登録できます。

[Firecrawl ホームページ](https://www.firecrawl.dev/referral?rid=W385F95R)

※ 記事中の Firecrawl へのリンクの一部は紹介リンク（アフィリエイトリンク）です。

[^pr]: [Firecrawl Raises $75M Series B; Launches Alexandria（PR.com, 2026-09-22）](https://www.pr.com/press-release/979737)
[^blog]: [Introducing Alexandria and our $75M Series B（Firecrawl Blog, 2026-09-22）](https://www.firecrawl.dev/blog/introducing-alexandria-series-b)
[^pricing]: [Firecrawl Pricing](https://www.firecrawl.dev/pricing)

## 参考

- [Alexandria（Firecrawl Docs）](https://docs.firecrawl.dev/features/alexandria)
- [Node.js SDK（Firecrawl Docs）](https://docs.firecrawl.dev/sdks/node)
