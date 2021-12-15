---
title: "Bootstrap のカスタマイズ (カラー、関数)"
emoji: "🐸"
type: "tech"
topics: ["CSS", "Bootstrap"]
published: false
---

:::details メモ: Bootstrap 4 のカスタマイズ

* 変数設定ファイル _variables.scss を _custom.scss でオーバーライドする仕組み
    - bootstrap.scss で、_custom.scss, _variables.scss の順にインポートされる
    - _variables.scss の各プロパティには !default が付いているため、_custom.scss に同名変数をセットすることで上書き
    - デフォルトでは node_module 内の _custom.scss が読み込まれる
    - プロジェクト内の任意の場所に _custom.scss をコピーし上書きしたい設定を書き、bootstrap.scss で読み込んでいる _custom.scss のパスをプロジェクト内のものを読み込ませるよう修正。これで Bootstrap をアップデートしても設定が残る。外部読み込みせず、直接変数を記述してもよい
* アップデートでデフォルトの変数自体が増減する可能性があるので、アップデートごとにチェックする作業は必要
:::


## Bootstrap (v5) グローバル CSS 変数の仕組みを知る

[CSS variables · Bootstrap v5.1](https://getbootstrap.com/docs/5.1/customize/css-variables/#root-variables)  
Root variables (node_modules/bootstrap/scss/_root.scss)

> v5.1.1では、すべてのCSSバンドル（bootstrap.css、bootstrap-reboot.css、bootstrap-grid.cssを含む）に必要な@importを、_root.scssを含むように標準化しました。これにより、バンドル内で使用されている変数の数に関わらず、すべてのバンドルに :root レベルの CSS 変数が追加されます。最終的にBootstrap 5では、時間の経過とともに、より多くのCSS変数が追加されていきます。\
  <cite>[Reboot - Bootstrap v5.1](https://getbootstrap.com/docs/5.1/content/reboot/#css-variables)</cite>

* 上の一覧の他にいくつかのコンポーネントで CSS カスタムプロパティが使われている
    * 2021年10月現在：table, __grid.scss (row,gap)
        * _grid.scss 中はデフォルト値がセットされる。以下の通りなので変更は不要
            * カラム数やセル間隔(gap)は `--#{$variable-prefix}columns, #{$grid-columns};` と変数をセットするようになっている
            * grid-template-rows: 1
        * _tables.scss も、変数をセットするようになっているので変更不要
        * _modal.scss, helpers/_position.scss 等のコンポーネントでも読み込んでいる
    * ※上記の通り、増えていく予定


## カスタマイズする前に必要な知識

* _variables.scss で Bootstrap のデフォルトのスタイル・挙動が設定されている
* ↑を style.scss の変数を上書きすることでカスタマイズできる
	* `$primary: #6a1816; // _variables.scss の #0d6efd($blue)から変更`
* _root.scss で、CSSグローバル変数として設定される
    * --bs-primary: #6a1816;
    * :root 疑似要素にセットされ、ドキュメント全体で有効
    * JavaScript からスタイルを参照できる
    * プレフィックスの変更方法
        * デフォルトで "bs-" → --bs-primary
        * 変更：style.scss に `$variable-prefix: hoge-;` を追記で --hoge-primary になる
    * CSSグローバル変数 mixin の作り方
        * [Tables · Bootstrap v5.1](https://getbootstrap.com/docs/5.1/content/tables/#how-do-the-variants-and-accented-tables-work)
    * _root.scss に設定されているもの全て
      - $colors、$grays、$theme-colors、$theme-colors-rgb をループで
        - white-rgb、black-rgb、body-color-rgb、body-bg-rgb
        - font-sans-serif、font-monospace、gradient
        - root-font-size (@if $font-size-root != null)
        - body-font-family、body-font-size、body-font-weight、body-line-height、body-color
        - body-text-align (@if $body-text-align != null)
        - body-bg
    * CSS/Sass で使用
        * `.hoge {var(--hoge-primary);`


## カラーの追加、削除、編集 (Color Sass maps)

[Color · Bootstrap v5.1](https://getbootstrap.com/docs/5.1/customize/color/)

"基本カラー" と "テーマカラー" がある。  
設置場所に注意。

```scss
// Required (後のコードで使われる変数や関数があるので先に読み込む)
@import "../node_modules/bootstrap/scss/functions";
@import "../node_modules/bootstrap/scss/variables";
@import "../node_modules/bootstrap/scss/mixins";
@import "../node_modules/bootstrap/scss/root";

//
// ここに記述
//

// Optional
@import "../node_modules/bootstrap/scss/reboot";
@import "../node_modules/bootstrap/scss/type";
// etc...
@import "../node_modules/bootstrap/scss/utilities/api";
```

### 基本カラー

$colors で設定

:::message
デフォルト基本カラー："blue", "indigo", "purple", "pink", "red", "orange", "yellow", "green", "teal", "cyan", "white", "gray", "gray-dark
:::

#### 既存の基本カラーの変更

変数で上書き
`$red: #FF3333;`

#### 不要なものを削除

```scss
// "red", "yellow" 以外を削除
$colors: map-remove($colors, "blue", "indigo", "purple", "pink", "orange", "green", "teal", "cyan");
```
#### 基本カラーの追加

```scss
// Create your own map (追加するカラー)
$custom-colors: (
  "maroon": #800000
);

// Merge the maps ($colors に統合)
$colors: map-merge($colors, $custom-colors);
```

### テーマカラー

$theme-colors で設定

:::message
デフォルトテーマカラー："Primary","Secondary","Success","Danger","Warning","Info","Light","Dark"
:::

#### 既存のテーマカラーの変更

変数で上書き
`$primary: #0074d9;`

#### 不要なものを削除

```scss
// "success", "info" を削除
$theme-colors: map-remove($theme-colors, "success", "info");
```

#### テーマカラーの追加

```scss
// Create your own map (追加するカラー)
$custom-theme-colors: (
  "tertiary"     : $tertiary,
  "marker-color" : $marker-color
);

// Merge the maps ($theme-color に統合)
$theme-colors: map-merge($theme-colors, $custom-theme-colors);
```

### Generating utilities

Utility API で、ユーティリティクラスを追加できる。
scss/utilities/api を「後で」読み込むこと。

* [Utility API · Bootstrap v5.1](https://getbootstrap.com/docs/5.1/utilities/api/)
* [Color · Bootstrap v5.1](https://getbootstrap.com/docs/5.1/customize/color/#generating-utilities)


red, primary の color, background-color, border-color の明度スケールのカラー・ユーティリティクラスを作成するサンプル。

:::message
他のユーティリティやプロパティでも同様にできる。
:::

* .text-{color}-{level} (例 .text-primary-200) 
* .bg-{color}-{level} (例 .bg-red-700) 
* .border-{color}-{level} (例 .border-primary-100) 

:::メモ：CSS カスタムプロパティ、calc()、hsl() 関数で実装する方法
Sass なしで実装できる。でも、10段階のカラースケール用途だと、圧迫感のあるコードになりそう。  
[How to use HSL and calc functions in CSS to build darken and light colors - DEV Community](https://dev.to/danywalls/how-to-use-hsl-and-calc-functions-in-css-for-creating-darken-and-light-colors-3kbn)
:::

```scss
// Primary カラーのスケールを作成
$primary-100   : tint-color($primary, 80%);
$primary-200   : tint-color($primary, 60%);
$primary-300   : tint-color($primary, 40%);
$primary-400   : tint-color($primary, 20%);
$primary-500   : $primary;
$primary-600   : shade-color($primary, 20%);
$primary-700   : shade-color($primary, 40%);
$primary-800   : shade-color($primary, 60%);
$primary-900   : shade-color($primary, 80%);

// 配列にセット
$primarys: (
  "primary-100": $primary-100,
  "primary-200": $primary-200,
  // ...
  "primary-500": $primary-500, // = $primary
  // ...
  "primary-900": $primary-900
);

// 適用したいカラーの配列を Sassマップにマージ
$all-colors: map-merge-multiple(
  $reds, // デフォルト基本カラーは、元からスケールが用意されている
  $primarys
);

// color, background-color, border-color に適用
$utilities: map-merge(
  $utilities,
  (
    "color": map-merge(
      map-get($utilities, "color"),
      (
        values: map-merge(
          map-get(map-get($utilities, "color"), "values"),
          (
            $all-colors
          ),
        ),
      ),
    ),
    "background-color": map-merge(
      map-get($utilities, "background-color"),
      (
        values: map-merge(
          map-get(map-get($utilities, "background-color"), "values"),
          (
            $all-colors
          ),
        ),
      ),
    ),
    "border-color": map-merge(
      map-get($utilities, "border-color"),
      (
        values: map-merge(
          map-get(map-get($utilities, "border-color"), "values"),
          (
            $all-colors
          ),
        ),
      ),
    ),
  )
);
```