---
title: "Cloudflare の画像配信は「制約」から逆算すると R2 + wasm-vips に着地する"
emoji: "🐸"
type: "tech"
topics: ["cloudflare", "r2", "workers", "image", "typescript"]
published: false
---

AI Deck という AI サービスブックマークアプリを開発しているのですが、投稿機能を作っていて、ある日悩みに直面しました。

「本文に画像を埋めたい。ついでにレスポンシブ画像にもしたい。でも、ストレージどこに置く？」

R2、Cloudflare Images、S3 互換、Cloudinary、`public/images` への静的配置。選択肢は山ほどあります。料金も実装コストも運用負荷もバラバラで、どれを選ぶか迷いました。同じ悩みを抱えている方は多いのではないでしょうか。

結論から言うと、R2 + wasm-vips + react-markdown のレンダラー拡張で落ち着きました。この記事では、そこに至るまでに整理した **2 つの決定的な制約** と、**4 つの選択肢の比較過程** をまとめます。

「選択肢を並べて比べる」のではなく、**「制約から逆算する」**。これが、画像配信に限らず技術選定全般で判断をブレさせないコツです。

## 前提: 何を解きたかったか

AI Deck は Cloudflare Workers + D1 + Pages で動いているウェブアプリで、投稿本文は Markdown を textarea で直接書く方式です。

必要だったのは 3 つ。

1. 管理者がエディタから画像をアップロードして、即 Markdown に挿入できる
2. 配信はレスポンシブ対応（スマホに 1920px の元画像を送らない）
3. 長期運用でもシンプルさを保ちたい

「まず R2 でいいでしょう」で進めてもいいのですが、選択肢を一度整理しておかないと、後から設計変更が発生します。というわけで、制約から逆算します。

## 制約 1: Cloudflare Pages は書き込めない

これが最も大きい制約でした。

Cloudflare Pages は静的ファイル配信専用で、ランタイムでファイルシステムに書き込めません。つまり `apps/web/public/images/` に動的にファイルを追加する選択肢は **ゼロ** です。

「じゃあ静的配置を諦めるしかないの？」と思うかもしれません。実は、事前に画像をローカルで用意 → Git commit → push → Pages 再ビルド、という手順でなら可能です。

でも、これで記事を 1 本書くたびに以下が必要になります。

- ローカルで画像を用意
- `public/images/posts/xxx.jpg` に配置
- 複数サイズ版も用意（レスポンシブ用）
- `git add` `git commit` `git push`
- Pages の再ビルドを 1〜3 分待つ
- URL を手書きで Markdown に貼る

これで成立するのは「画像を月に数枚しか追加しない」運用です。AI Deck のように記事を継続追加するなら、このワークフローは破綻します。

加えて、Git にバイナリが蓄積される問題もあります。複数サイズ × 記事数 × 差し替え回数で、リポジトリが継続的に肥大化。画像差し替えで `git history` が汚れ、将来別リポジトリに移行するときに重荷になります。

**結論**: Pages の書き込み不可制約により、動的アップロード UI が欲しいなら静的配置は自動的に却下です。

## 制約 2: Markdown にはレスポンシブ画像が書けない

もうひとつ、意外と見落とされる制約がこれです。

CommonMark / GFM の画像構文は `![alt](url "title")` のみ。吐き出せる属性は `src`・`alt`・`title` の 3 つだけ。**srcset も sizes も picture 要素も書けません**。

これが意味するところは大きいです。どのストレージを選ぼうが、Markdown に `![](xxx.jpg)` と書いた瞬間、出力は `<img src="xxx.jpg">` に固定されます。スマホに 1920px 元画像をそのまま送ってしまいます。

「じゃあ HTML で `<img srcset="...">` を書けばいい」と思いますよね。確かに `rehype-raw` を使えば Markdown 内に生 HTML を埋められます。でも、実際に書くとこうなります。

````markdown
## セクションタイトル
説明文です。

<picture>
  <source srcset="/images/posts/xxx-640.avif 640w, /images/posts/xxx-1024.avif 1024w" type="image/avif">
  <source srcset="/images/posts/xxx-640.webp 640w, /images/posts/xxx-1024.webp 1024w" type="image/webp">
  <img src="/images/posts/xxx-1024.jpg"
       srcset="/images/posts/xxx-640.jpg 640w, /images/posts/xxx-1024.jpg 1024w, /images/posts/xxx-1920.jpg 1920w"
       sizes="(max-width: 640px) 100vw, 800px"
       alt="..." loading="lazy">
</picture>

続きの文章。
````

これが 1 記事に 10 個散在すると、後から自分で修正するときに消耗します。Markdown の「軽量マークアップ」という本来のメリットが消滅してしまいます。

ではどうするか。**レンダラー側で補う** のが王道です。react-markdown の `components` プロップで `img` 要素の変換処理を差し替えられます。

```tsx
<ReactMarkdown
  components={{
    img: ({ src, alt }) => {
      if (src?.startsWith("https://images.ai-deck.app/")) {
        const pathname = new URL(src).pathname;
        const base = pathname.replace(/-(\d+)\.webp$/, "");
        return (
          <img
            src={`https://images.ai-deck.app${base}-1024.webp`}
            srcSet={[640, 1024, 1920]
              .map(w => `https://images.ai-deck.app${base}-${w}.webp ${w}w`)
              .join(", ")}
            sizes="(max-width: 640px) 100vw, 800px"
            alt={alt}
            loading="lazy"
          />
        );
      }
      return <img src={src} alt={alt} loading="lazy" />;
    },
  }}
>
  {content}
</ReactMarkdown>
```

書き手は `![](xxx-1024.webp)` のまま、出力は自動で srcset 付き `<img>`。この仕組みを前提に、ストレージを選ぶ必要があります。

**結論**: Markdown 単体ではレスポンシブ画像は書けません。レンダラー拡張は、どの選択肢を選ぼうが必須になります。

## 選択肢の比較

2 つの制約が固まったので、選択肢を絞り込みます。

| 方式 | UI アップ | Git 影響 | 月額 | 実装時間 | 備考 |
|---|---|---|---|---|---|
| public/images 静的配置 | ❌ | 大（履歴肥大） | $0 | 1h | Pages 制約で自動却下 |
| Cloudflare Images | ✅ | なし | $5 | 2-3h | マネージド、variants 自動切替 |
| Cloudflare Image Resizing | ✅ | なし | $20 (Pro 必須) | 4h | R2 + `/cdn-cgi/image/` で動的リサイズ |
| R2 + wasm-vips | ✅ | なし | $0 (R2 ストレージ $0.x) | 8-16h | Worker 内で自前変換 |

Pro プラン $20/月 は AI Deck には過剰なので、Image Resizing はまず除外。残るは 3 つです。

### Cloudflare Images の魅力

- **アカウント単位契約**で、同じ Cloudflare アカウントの別サイトからも利用できる（追加料金なし、利用量が枠内なら）
- variants を後から追加すると既存全画像に即適用
- AVIF/WebP の自動判別がマネージド
- 実装は 2-3 時間で完了

引っかかる点もあります。

- 月額 $5 が固定で発生（利用量ゼロでも）
- 配信ドメインが `imagedelivery.net/<hash>/...` になり、独自ドメイン化には Pro プラン ($20) が必要

### wasm-vips の魅力

- ランニングコストは R2 ストレージ分のみ ($0.x/月)
- Worker 内で完結、自分のインフラで完結
- 将来、透かし追加や特殊フィルタなどカスタム処理を入れたくなった時の自由度

罠もあります。

- Worker bundle size 制限（圧縮後 3MB）にかなり迫る（wasm-vips は 2-3MB）
- 大きい画像のリサイズは数秒かかる。10MB のフル解像度写真を 5 サイズ変換すると CPU 時間を圧迫する
- エッジケースのエラーハンドリングが増える
- ローカル開発での Miniflare との相性確認が必要

### 時給換算で総コストを比較

時給 $50/h として、3 年総コストを並べてみます。

| | 実装時間コスト | 36 ヶ月ランニング | 合計 |
|---|---|---|---|
| Cloudflare Images | $100-150 | $180 | **$280-330** |
| wasm-vips | $400-800 | $5.4 | **$405-805** |

金額だけ見ると Cloudflare Images が安いです。これが合理的な比較結果。

## でも wasm-vips を選びました

数字では Cloudflare Images の勝ちなのですが、最終的には wasm-vips + R2 を選びました。理由を 5 つ挙げます。

### 1. サブスクの心理コスト

月 $5 という金額は小さいです。でも「新しいサブスクを 1 件増やす」という意思決定の心理コストは意外に重いものがあります。管理するサブスクが増えれば増えるほど、全体の可視性が落ちます。

Cloudflare Workers Paid ($5/月) は既に払っています。R2 は従量課金で $0.x/月。wasm-vips は追加コストゼロ。つまり **既存の 1 サブスクで完結** します。これは長期的な精神衛生上のメリットが大きいです。

### 2. 配信ドメインの一貫性

Cloudflare Images は `imagedelivery.net/<hash>/...` で配信されます。独自ドメイン化には Pro プラン ($20/月) が必要です。

R2 は Custom Domain が無料で接続できます。`images.ai-deck.app` で配信できるので、CSP 設定も CDN キャッシュも見通しがシンプルです。ブランディング上も、配信 URL が自社ドメインのほうが気持ちが良いです。

### 3. スタック親和性

Cloudflare Workers + R2 は binding 1 行で繋がります。API レイヤーで `env.UPLOADS_BUCKET.put(key, bytes)` と書くだけ。サービス境界がほぼ存在しません。

Cloudflare Images も同じアカウント内ですが、API アクセスは別 endpoint (`api.cloudflare.com/.../images/v1`)、配信も別ドメイン。境界がひとつ増えます。細かい話に見えますが、エラー発生時の原因切り分けや、権限管理の複雑度に効いてきます。

### 4. 将来の拡張性

記事本文に埋め込む画像だけでなく、将来こういう要件が出てきそうです。

- OG 画像の動的生成（記事タイトルから自動合成）
- サムネイルへの文字合成
- 図表の自動キャプション追加

こうした処理が出てきたとき、wasm-vips なら同じ Worker 内で完結できます。Cloudflare Images は定型処理は得意ですが、ドメイン特化のカスタムロジックは不得意です。

### 5. 諦めたもの（正直に書く）

実装時間で 4-8 時間余計に使います。wasm-vips のバージョンアップ追従や、Worker bundle size の監視も自分で面倒を見ることになります。

これを受け入れられるかどうかが分かれ目です。私の場合「画像配信は今後 3 年以上継続運用する」「技術的な自由度が長期的に効く」という判断で受け入れました。

逆に、開発リソースが限られていて「実装して運用に乗せるまでの時間」を最短化したいなら、Cloudflare Images で 2-3 時間で終わらせるほうが合理的です。

## 実装の勘所

採用構成の全体像です。

```
[Admin UI] → POST /api/admin/uploads (Cloudflare Worker)
                 ↓
            wasm-vips で 3 サイズ生成 (640, 1024, 1920) + WebP 変換
                 ↓
            UPLOADS_BUCKET.put(key, bytes) × 3
                 ↓
            URL 返却: https://images.ai-deck.app/posts/.../xxx-1024.webp

[記事閲覧] → MarkdownContent の components.img が srcset 自動生成
              → <img srcset="... 640w, ... 1024w, ... 1920w" sizes="...">
```

### R2 のキー設計

```text
posts/{YYYY-MM}/{uuid}-{size}.webp
  例: posts/2026-04/a1b2c3d4-640.webp
      posts/2026-04/a1b2c3d4-1024.webp
      posts/2026-04/a1b2c3d4-1920.webp
```

月別プレフィックスにしておくと、後から list 操作（孤立画像の清掃 Cron など）がやりやすくなります。UUID は衝突回避と推測不能性の両立のためです。

### Worker での wasm-vips 運用 Tips

実装時にハマりやすいポイントを先出ししておきます。

- **初期化コスト**: `await Vips()` は呼ぶたびに初期化が走る。ウォームスタートで使い回す工夫が必要
- **サイズ変換**: `thumbnail_buffer` を使うと内部最適化されて高速
- **WebP 変換**: `webpsave_buffer` で `Q=80` 程度が視覚的品質とファイルサイズのバランスが良い
- **メモリ管理**: 変換後の画像オブジェクトは早めに `delete()` を呼ばないとメモリ不足になる
- **マジックナンバー検証**: アップロード時に偽装ファイル対策で先頭バイトを確認

### レンダラー拡張（再掲）

`MarkdownContent.tsx` の `components.img` を上書きするだけ。書き手（自分）は Markdown のシンプルさを失わず、配信時に自動で最適化されます。1 箇所の設定で、全記事の全画像に適用される点が大きいメリットです。

`sizes` 属性を統一で管理できるのも利点です。「`sizes` を 800px から 1024px に変えたい」と思ったとき、この 1 箇所を修正するだけで全記事に反映されます。

## 選定は「金額」ではなく「制約」で決まる

この記事で伝えたかったのは、この一点です。

画像配信のストレージ選定をするとき、「R2 は安い」「Images は楽」という各選択肢の「良さ」を並べても決まりません。それぞれに得意な場面があるからです。

代わりに、先に **制約を書き出してみます**。

- Pages は書き込めない → 静的配置の UI アップロード選択肢は自動的に消える
- Markdown は srcset を書けない → レンダラー拡張は必須、どの選択肢でも共通実装
- サブスクは心理コスト → 既存サブスクで完結するなら優先度アップ
- 独自ドメイン化 → Images は Pro プラン必要、R2 は無料

この 4 つを整理すると、残る選択肢が自動的に 1-2 個に絞られます。あとはスタック親和性や将来の拡張性で選ぶだけです。

選択肢を並べて比較するのは、時間がかかる割に判断がブレがちです。制約から逆算すると、比較軸そのものが整理されて、結論のブレがなくなります。

画像配信に限らず、技術選定全般に使える考え方です。

次に何かを選ぶとき、まず制約を書き出してみてください。意外と、選ぶべきものが最初から決まっていたことに気づくかもしれません。
