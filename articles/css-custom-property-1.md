---
title: "CSS カスタムプロパティ 1 基礎"
emoji: "🐸"
type: "tech"
topics: [CSS]
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


:::details CSS カスタムプロパティに対応してないブラウザでフォールバック
### Polyfill

[IE11 Polyfill](https://github.com/nuxodin/ie11CustomProperties) サポート終了

```html
<script>window.MSInputMethodContext && document.documentMode && document.write('<script src="https://cdn.jsdelivr.net/gh/nuxodin/ie11CustomProperties@4.1.0/ie11CustomProperties.min.js"><\/script>');</script>
```

### CSS カスタムプロパティに対応してないブラウザでフォールバックする方法

でも、[@supports -MDN](https://developer.mozilla.org/ja/docs/Web/CSS/@supports) が IE11 に対応していないので、この場合での使いどころはなさそう

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

:::details env() 関数でのフォールバック
### フォールバック

[env() - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/env()) 文書でグローバルなデフォルト値を設定し、その値をフォールバックとして使える。\
例：`env(safe-area-inset-top, 20px);`\
しかし、パフォーマンスが落ちるとなにかで読んだ気が。。。(探したが見つからず)
:::

### 無効な変数の場合

* プロパティの初期値または継承値が使用される

### インラインCSSで上書きできる

* `<p class="primary" style="--primary-color: gold;"></p>`

### JavaScript で取得、値をセット

* `:root` 疑似クラスで設定した値は、`document.documentElement` でアクセスできる
    - 指定カスタムプロパティの値を取得
        * `document.documentElement.style.getPropertyValue('--primary');` ダメ
            - TODO: 確認：インラインCSSなら、getPropertyValue だけで取得できる?  
             [Access CSS variable from javascript - Stack Overflow](https://stackoverflow.com/questions/41725725/access-css-variable-from-javascript)
        * `window.getComputedStyle(document.documentElement).getPropertyValue('--primary');` OK
    - 指定カスタムプロパティに値をセット
        * `document.documentElement.style.setProperty('--primary', "pink");` 元々ないものも追加可能

* ブラウザ開発ツールで要素を選択し、"Dom プロパティを表示" で確認できる
  → Style: CSS2Properties

### デメリットを感じる点

* `var(--)` って打つのがめんどい
* エディタのコード横カラー表示がない
* 複雑になると見通しが悪い (特に calc() 関数などを使うと)

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

### calc() 関数と併せて使う 

[デモ3](https://jsfiddle.net/takna/tj74suzd/)

* [calc() - MDN](https://developer.mozilla.org/ja/docs/Web/CSS/calc())
* カラーを HLS で指定すれば、カスタムプロパティで色を自在にコントロールできる
    - [hsl() - MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/color_value/hsl())
    - 各指定値をカスタムプロパティ化すると、コントラスト色、明度・彩度スケールなど色々できるが、見通しは悪くなるので使用は絞った方が良さげ
    - 参考：[Creating Color Themes With Custom Properties - CSS-Tricks](https://css-tricks.com/creating-color-themes-with-custom-properties-hsl-and-a-little-calc/)

### CSS アニメーションで

今回は割愛


## まとめ

* 素のCSSの全てのプロパティで使えるので、アイデア次第で色々できる
    - 色・透明度、サイズ、余白、レイアウト(グリッド他)、表示/非表示、アニメーション