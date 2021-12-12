---
title: "CSS カスタムプロパティ 1 基礎"
emoji: "🐸"
type: "tech"
topics: ["CSS"]
published: true
---


## CSS カスタムプロパティ(CSS変数) とは

[デモ1](https://jsfiddle.net/takna/c32uhkrg/)

* `:root` 擬似クラスは、html 要素に適用される
* 使用したい箇所に `var()` 関数で指定
* 大文字と小文字を区別するので注意
* 変数のように扱えるので、複数箇所のスタイルを一元管理するのに便利
* CSSカスタムプロパティと呼ばれることが多い。値を格納する変数ではなく動的に反映されるプロパティ
* [対応ブラウザ](https://caniuse.com/css-variables) IE切るなら使える

### Sass(SCSS)等の CSSプリプロセッサーとの違い

* ブラウザ上で変数のままで動的に処理される。JavaScript から参照、変更できる
* Sass (scss) の場合、トランスパイルが必要で、変換後は固定値となる

### Sass 中でも使える

* カスタムプロパティを Sass 変数に格納するのは無理。エラーに

### details CSS カスタムプロパティに対応してないブラウザでフォールバックする方法

:::details Polyfill, @supports
### Polyfill

[IE11 Polyfill](https://github.com/nuxodin/ie11CustomProperties) サポート終了

```html
<script>window.MSInputMethodContext && document.documentMode && document.write('<script src="https://cdn.jsdelivr.net/gh/nuxodin/ie11CustomProperties@4.1.0/ie11CustomProperties.min.js"><\/script>');</script>
```

### @supports

[@supports -MDN](https://developer.mozilla.org/ja/docs/Web/CSS/@supports) で以下のようにできるが、肝心の IE11 に対応していないので、このケースでの使いどころはなさそう

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
:::

### デフォルト値

* 最初のカンマから関数の終わりまでが代替値とみなされる
* `var(--my-var, var(--my-background, pink));` のようにネストできるが、パフォーマンスが落ちるので避ける

#### env() で文書でグローバルなスコープのデフォルト値を設定できる

[env() - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/env()) 文書でグローバルなデフォルト値を設定できる。\
例：`env(safe-area-inset-top, 20px);`\

### 無効な変数の場合

* 継承値、なければプロパティの初期値が適用される

### インラインCSSで上書きできる

* `<p class="primary" style="--primary-color: gold;"></p>`

### JavaScript で取得、値をセット

* 従来のCSSプロパティの扱いと同じ
* `:root` 疑似クラスで設定した値は、`document.documentElement` でアクセスできる
- 指定カスタムプロパティの値を取得
    * `window.getComputedStyle(document.documentElement).getPropertyValue('--primary');`
        - メモ：`document.documentElement.style.getPropertyValue('--primary');` では取得できなかった [Access CSS variable from javascript - Stack Overflow](https://stackoverflow.com/questions/41725725/access-css-variable-from-javascript), [Window.getComputedStyle() - MDN](https://developer.mozilla.org/ja/docs/Web/API/Window/getComputedStyle) 違いを解説
- 任意のカスタムプロパティに値をセット
    * `document.documentElement.style.setProperty('--color-pink', "pink");` 元々ないものも追加可能
      - この場合は、html 要素にインラインで埋め込まれる  
        `<html style="--color-pink: pink;">`
* メモ：JavaScript で操作したものは、ブラウザ開発ツールでHTML要素を選択、"Dom プロパティを表示" > "Style: CSS2Properties" で、カスタムプロパティを確認できる

### その他メモ

* `var(--)` と打つのがめんどい (→ コードスニペットで展開)
* 複雑になると見通しが悪い (特に calc() 関数などを使った場合)  
* エディタの補完や表示のサポートはまちまち

[//]: # (* phpstorm &#40;intellij 系&#41; なら、var&#40;--primary&#41; で色チップが表示)

[//]: # (* VSCode [CSS Custom Properties - Visual Studio Marketplace]&#40;https://marketplace.visualstudio.com/items?itemName=Tock.vscode-css-custom-properties&#41;)

---

#### メディアクエリで

[デモ2](https://jsfiddle.net/takna/yq9hwz30/)

* メディアクエリ規則の中では使えない。    
  `@media (max-width: var(--mobile-breakpoint)) {}` は効かない。
* メディアクエリ分岐で全体に影響するカスタムプロパティの値を変えたい場合は、:root をメディアクエリでスコープ分岐させる。
    - [CSS Variables - Google Developers](https://developers.google.com/web/updates/2016/02/css-variables-why-should-you-care)
* レスポンシブデザインで、サイズや余白調整に便利

---

### スコープ

[デモ3](https://jsfiddle.net/takna/40yngfkb/)

* `:root` 疑似クラスで文書全体がスコープに (≒グローバル変数)
* プログラム言語の関数みたいな感じ
  - 内側スコープ内を検索しあれば適用。無ければ一つ親の要素を探す。無ければそのさらに親の… :root まで探して無ければ無効値 (ブラウザ初期値) に
* 従来のCSSの継承 (カスケード) そのまま

---

### CSS関数と併せて使う 

[デモ3](https://jsfiddle.net/takna/tj74suzd/)

* [calc() - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/calc())
* カラーを HLS で指定すれば、カスタムプロパティで色を自在にコントロールできる
    - [hsl() - MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/hsl())
    - 各指定値をカスタムプロパティ化すると、コントラスト色、明度・彩度スケールなど色々できるが、見通しは悪くなるので使用は絞った方が良さげ
    - 参考：[Creating Color Themes With Custom Properties - CSS-Tricks](https://css-tricks.com/creating-color-themes-with-custom-properties-hsl-and-a-little-calc/)
* [max()](https://developer.mozilla.org/ja/docs/Web/CSS/max()) [clamp()](https://developer.mozilla.org/ja/docs/Web/CSS/clamp()) [min()](https://developer.mozilla.org/ja/docs/Web/CSS/min()) 関数で最小値、推奨値、最大値を設定できる


### CSS アニメーションで、レイアウトで

今回は割愛


### フレームワーク、CSS、ライブラリ等を使う場合

#### 冗長にならないよう注意

それらが指定する CSS カスタムプロパティを把握した上で実装すること。

#### プレフィックスを付ける

もし読み込んでいる他のCSSのカスタムプロパティ名と重複すると、不具合が起きるので、このプロジェクトで統一したプレフィックスを付けとく方が無難な場合がある。
`--primary` → `--XX-primary`
(WordPress は "--wp--preset--"、Bootstrap は "--bs-" と付いてる)

Sass なら、`$prefix: hoge;` `--#{$prefix}-primary-color: red;` みたいにすれば後から容易に変更できる

## まとめ

* 素のCSSの全てのプロパティで使える、また、JavaScript で操作できるので、アイデア次第で色々できる
    - 色・透明度、フォント、サイズ、余白、レイアウト(グリッド他)、表示/非表示、アニメーション

## 参考記事

* [CSSの変数（カスタムプロパティ）便利な使い方を詳しく解説 | コリス](https://coliss.com/articles/build-websites/operation/css/css-variables.html)
    - [CSSの変数（カスタムプロパティ）が期待通りに動作しないときの解決方法 | コリス](https://coliss.com/articles/build-websites/operation/css/solution-when-custom-properties-do-not-work.html)
* [CSSで変数（カスタムプロパティ）を使ってみよう - Webクリエイターボックス](https://www.webcreatorbox.com/tech/css-variables)
* [Patterns for Practical CSS Custom Properties Use - CSS-Tricks](https://css-tricks.com/patterns-for-practical-css-custom-properties-use/) Grit など
* [A Guide To Modern CSS Colors With RGB, HSL, HWB, LAB And LCH — Smashing Magazine](https://www.smashingmagazine.com/2021/11/guide-modern-css-colors/) HSLカラー
* [How to create better themes with CSS variables - LogRocket Blog](https://blog.logrocket.com/how-to-create-better-themes-with-css-variables-5a3744105c74/) サンプルいっぱい
* [A Complete Guide to Custom Properties - CSS-Tricks](https://css-tricks.com/a-complete-guide-to-custom-properties/) サンプル、リンクいっぱい
  - [Exploring @property and its Animating Powers - CSS-Tricks](https://css-tricks.com/exploring-property-and-its-animating-powers/) まだ対応ブラウザ限られる