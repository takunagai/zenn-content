---
title: "CSS カスタムプロパティ 2 カラー設定に使う (HSL, calc)"
emoji: "🐸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["CSS"]
published: false
---

## HSL カラーの基本

★★

### calc() 関数 と HSL との組み合わせでカラー自動設定

カラーで HSL (色相, 彩度, 明度) を使うことで、

calc() を使うことで、計算結果の代入も可能。
色相, 彩度, 明度をずらすことで、システマチックに色をコントロールできる (反対色、色薄めにとか)。

```css
<style>
  .card { /* カードコンポーネント共通 */
    --th:  hsl(0, 0%, var(--lightness, 70%));
    --accent: #5c290a;
    background: var(--th);
  }
  .card--event { /* 優先される */
    --th:  hsl(72, 67%, var(--lightness, 70%));
  }
  .card:hover {
    --lightness: 80%;
  }
  .card__heading a {
    color: var(--accent);
  }
</style>

<div class="card">
  <h3 class="card__heading"><a href="#">タイトル</a></h3>
  <p class="card__body">ダミーコピーです手はおっかさんの演奏硝子屋をセロに思ったばこだた。</p>
</div>
<div class="card card--event">
  <h3 class="card__heading"><a href="#">タイトル2</a></h3>
  <p class="card__body">ダミーコピーです手はおっかさんの演奏硝子屋をセロに思ったばこだた。</p>
</div>
```

```css
.card {
  --hue: 259;
  --hueComplementary: calc(var(--hue, 20) + 180); // 反対色用(色相180度ずらす)
  --th: hsl(var(--hue), 72%, 64%);
  --thDark: hsl(var(--hue), 72%, 38%);
  --accent: hsl(var(--hueComplementary, var(--hue)), 72%, 64%);
}
```