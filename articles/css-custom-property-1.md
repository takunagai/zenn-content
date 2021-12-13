---
title: "CSS カスタムプロパティ 1 基礎"
emoji: "🐸"
type: "tech"
topics: ["CSS"]
published: true
---


## CSS カスタムプロパティ (CSS 変数) とは

素の CSS で、変数感覚で扱えるプロパティ。複数箇所のスタイルを一元管理するのに便利。また、Dom のプロパティなので、値が動的にブラウザ表示に反映されるため JavaScript から値をコントロールすることで、様々な有用な処理ができる。

[» 対応ブラウザ](https://caniuse.com/css-variables)  
※ IE 11 は非対応 (サポート終了の Polyfill はある)

:::message
"CSS 変数" とも呼ばれ、var() という関数を使用するが、値を格納する変数ではなくプロパティなので、"CSS カスタムプロパティ" と呼ぶ方が妥当。
:::

:::details 習得すべき？
すでに様々な CMS、フレームワーク、ライブラリ等で使われており、自分のプロジェクトの CSS を最適化するためにはそれらを考慮した実装が必要となる。素の CSS なので廃れて無くなることもない。使いこなすと便利なはずなので習得しておいて損はない。
:::

## 基本の基本

[デモ1](https://jsfiddle.net/takna/c32uhkrg/)

* `:root` 擬似クラスは、ドキュメント全体 (html 要素) に適用される
* 使用したい箇所で `var(--任意の名前)`
* 大文字と小文字を区別するので注意

### Sass と CSS カスタムプロパティー

#### Sass(SCSS)等の CSSプリプロセッサーとの違い

* ブラウザ上で変数のままで動的に処理される。なので、JavaScript から get/setPropertyValue で直接参照、変更できる
  - 従来はクラス切替えや setAttribute 関数での属性での指定での実装が多かった
* Sass (scss) の場合、トランスパイルが必要で、変換後は固定値となる

#### Sass 中でも使える

* 素のCSSなので、Sass でも普通に使える
* ~~カスタムプロパティを Sass 変数に格納するのは不可。エラーに~~

#### Sass は不要となるか？

ならない。変数や計算は代替できるが、ネスト、ミックスイン、継承、ファイルの統合などはやはり必要なので、Sass との併用が現実的。状況により、Sass の代わりに PostCSS、CSSinJS、Tailwind CSS 等を採用する選択肢もある。

### CSS カスタムプロパティに対応してないブラウザへの対応

:::details Polyfill, @supports, PostCSS プラグイン

### Polyfill

[IE11 Polyfill](https://github.com/nuxodin/ie11CustomProperties) サポート終了

```html
<script>window.MSInputMethodContext && document.documentMode && document.write('<script src="https://cdn.jsdelivr.net/gh/nuxodin/ie11CustomProperties@4.1.0/ie11CustomProperties.min.js"><\/script>');</script>
```

### @supports

[@supports -MDN](https://developer.mozilla.org/ja/docs/Web/CSS/@supports) で以下のようにできるが、肝心の IE11 に対応していないので、CSS カスタムプロパティのフォールバックとして使うことはないだろう

```scss
.my-component {
  background-color: #82f263;
}
@supports (--css: variables) {
  .my-component {
    --myVariable: #ef62e6;
    background-color: var(--myVariable);
  }
}
```

### PostCSS プラグイン

[postcss/postcss-custom-properties - Github](https://github.com/postcss/postcss-custom-properties)
変数を固定値として書き出し、サポートしているブラウザ向けに CSS カスタムプロパティを使った記述を書き出す。最終的な CSS が肥大化しそう
:::

### デフォルト値

#### var() 関数で

* `var(--primary, yellow);` --primary が無ければ yellow を指定
* 最初のカンマから関数の終わりまでが代替値とみなされる  
  `var(--hoge, aaa, bbb, ccc);` "aaa, bbb, ccc" が代替値
* `var(--my-var, var(--my-background, pink));` のようにネストできるが、パフォーマンスが落ちるので避ける

#### env() でグローバルなスコープのデフォルト値を設定

[env() - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/env()) 文書でグローバルなデフォルト値を設定できる。\
例：`env(safe-area-inset-top, 20px);`\

### 無効な変数の場合

* 継承値、なければプロパティの初期値が適用される

### インラインCSSで上書きできる

* `<p class="primary" style="--primary-color: gold;"></p>`

### JavaScript で値の取得/セット

* 従来の CSS プロパティの扱いと同じ
* `:root` 疑似クラスで設定した値は、`document.documentElement` でアクセスできる
* 実装
    - 任意のカスタムプロパティの値を取得
        * `window.getComputedStyle(document.documentElement).getPropertyValue('--primary');`
          - メモ：`document.documentElement.style.getPropertyValue('--primary');` では取得できなかった [Access CSS variable from javascript - Stack Overflow](https://stackoverflow.com/questions/41725725/access-css-variable-from-javascript), [Window.getComputedStyle() - MDN](https://developer.mozilla.org/ja/docs/Web/API/Window/getComputedStyle) 違いを解説
    - 任意のカスタムプロパティに値をセット
        * `document.documentElement.style.setProperty('--color-pink', "pink");` 元々ないものも追加可能
            - この場合は、html 要素にインラインで埋め込まれる  
            `<html style="--color-pink: pink;">`

:::message
メモ：JavaScript で操作したものは、ブラウザ開発ツールでHTML要素を選択、"Dom プロパティを表示" > "Style: CSS2Properties" で、カスタムプロパティを確認できる
:::

### その他 メモ

* エディタの補完や表示のサポートはまちまち
* `var(--)` と打つのがめんどい (→ コードスニペットで展開)

[//]: # (* phpstorm &#40;intellij 系&#41; なら、var&#40;--primary&#41; で色チップが表示)

[//]: # (* VSCode [CSS Custom Properties - Visual Studio Marketplace]&#40;https://marketplace.visualstudio.com/items?itemName=Tock.vscode-css-custom-properties&#41;)

---

#### メディアクエリで

[デモ2](https://jsfiddle.net/takna/yq9hwz30/)

* メディアクエリ規則の中では使えない    
  `@media (max-width: var(--mobile-breakpoint)) {}` は効かない
* メディアクエリ分岐で全体に影響するカスタムプロパティの値を変えたい場合は、:root をメディアクエリでスコープ分岐させる。
    - [CSS Variables - Google Developers](https://developers.google.com/web/updates/2016/02/css-variables-why-should-you-care)
* レスポンシブデザインで、サイズや余白調整に便利

---

### スコープ

[デモ3](https://jsfiddle.net/takna/40yngfkb/)

* `:root` 疑似クラスで文書全体がスコープに (≒グローバル変数)
* プログラム言語の関数のスコープと同じ
  - 内側スコープ内を検索しあれば適用。無ければ一つ親の要素を探す。無ければそのさらに親の… :root まで探して無ければ無効値 (ブラウザ初期値) に
* 従来のCSSの継承 (カスケード) そのまま

---

### CSS関数と併せて使う 

[デモ3](https://jsfiddle.net/takna/tj74suzd/)

* [calc() - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/calc())
* カラーを HLS で指定すれば、数値をセットした CSS カスタムプロパティで色を自在にコントロールできる
    - [hsl() - MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/hsl())
    - 各値をカスタムプロパティ化することで色々できるが、見通しは悪くなるので使用は絞った方が良さげ
    - 参考：[Creating Color Themes With Custom Properties - CSS-Tricks](https://css-tricks.com/creating-color-themes-with-custom-properties-hsl-and-a-little-calc/)
* [max()](https://developer.mozilla.org/ja/docs/Web/CSS/max()), [clamp()](https://developer.mozilla.org/ja/docs/Web/CSS/clamp()), [min()](https://developer.mozilla.org/ja/docs/Web/CSS/min()) 関数で最小値、推奨値、最大値を設定できる
    - [Fluid Type Scale in WordPress - Rich Tabor](https://richtabor.com/fluid-type-scale-theme-json/) レスポンシブ・フォントサイズでの使用例


### CSS アニメーションで、レイアウトで

今回は割愛

## 気をつけたいこと

### 複雑にし過ぎると見通しが悪くなる

入れ子(カスタムプロパティに別のカスタムプロパティの値をセット)にしたり、calc() 関数などを使った場合、複雑になりやすい。未来の自分や他人が見た時、最終的にどのような値になるのかを理解するのに労力を使うことになる。計算でいい感じにできることは多々あると思うが、あまり複雑になり過ぎないよう気をつけたい。

### フレームワーク、CSS、ライブラリ等を使う場合

#### 最適化する

フレームワーク、CMS、ライブラリ等が使用している CSS カスタムプロパティを把握した上で実装すること。例えば WordPress なら theme.json での設定で head タグ内にインラインで埋め込まれる CSS カスタムプロパティを把握し、同じ値は自分のCSSで値を直接定義せず、それらの CSS カスタムプロパティを使う。

#### プレフィックスを付ける

カスタムプロパティ名の重複はトラブルを招く可能性があるので、このプロジェクトで統一したプレフィックスを付けとくと無難。
`--primary` → `--XXX-primary`
(例えば、WordPress は "--wp-"、Bootstrap は "--bs-" (変更可) と付いてる)

メモ：Sass なら、`$prefix: hoge;` `--#{$prefix}-primary-color: red;` のようにすれば後から容易に変更できる

## まとめ

* アイデア次第で色々できる (下の参考記事の応用例を参照)
    - 色・透明度、フォント、サイズ、余白、レイアウト(グリッド他)、表示/非表示、アニメーション

## 参考記事

### わかりやすい解説

* [CSSの変数（カスタムプロパティ）便利な使い方を詳しく解説 | コリス](https://coliss.com/articles/build-websites/operation/css/css-variables.html)
    - [CSSの変数（カスタムプロパティ）が期待通りに動作しないときの解決方法 | コリス](https://coliss.com/articles/build-websites/operation/css/solution-when-custom-properties-do-not-work.html)
* [CSSで変数（カスタムプロパティ）を使ってみよう - Webクリエイターボックス](https://www.webcreatorbox.com/tech/css-variables)

### 応用例

* [Open Props: sub-atomic styles](https://open-props.style/) 汎用的に使えそうな CSS カスタムプロパティ 351個をセットしたライブラリ。参考用にも
* [Patterns for Practical CSS Custom Properties Use - CSS-Tricks](https://css-tricks.com/patterns-for-practical-css-custom-properties-use/) Grid など
* [A Guide To Modern CSS Colors With RGB, HSL, HWB, LAB And LCH — Smashing Magazine](https://www.smashingmagazine.com/2021/11/guide-modern-css-colors/) HSLカラー
* [How to create better themes with CSS variables - LogRocket Blog](https://blog.logrocket.com/how-to-create-better-themes-with-css-variables-5a3744105c74/) サンプルいっぱい
* [A Complete Guide to Custom Properties - CSS-Tricks](https://css-tricks.com/a-complete-guide-to-custom-properties/) サンプル、リンクいっぱい
  - [Exploring @property and its Animating Powers - CSS-Tricks](https://css-tricks.com/exploring-property-and-its-animating-powers/) @property は、まだ対応ブラウザ限られる