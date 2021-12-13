---
title: "CSS カスタムプロパティ 3 WordPress Theme.json"
emoji: "🐸"
type: "tech"
topics: ["CSS", "WordPress"]
published: false
---

[グローバル設定とスタイル (theme.json) – WordPress.org](https://ja.wordpress.org/team/handbook/block-editor/how-to-guides/themes/theme-json/#%E3%83%97%E3%83%AA%E3%82%BB%E3%83%83%E3%83%88)


### theme.json に設定

```theme.json
{
    "version": 1,
    "settings": {
        "color": {
            "palette": [
                {
                    "name": "Black",
                    "slug": "black",
                    "color": "#000000"
                },
                {
                    "name": "White",
                    "slug": "white",
                    "color": "#ffffff"
                }
            ]
        }
    }
}
```

```css
/* 出力される (theme.json でスコープを自由に設定できる。この場合は body がスコープ */
body {
    --wp--preset--color--black: #000000;
    --wp--preset--color--white: #ffffff;
}

/* 使用 */
.hoge { color: var(--wp--preset--color--primary); }
```


* カスタムの出力："--wp--custom--line-height--body: 1.7;"

* サイト内で一度に変更できる共有の値
* theme.json を作成することでコアのデフォルトを上書きできる

[グローバル設定とスタイル (theme.json) – WordPress.org](https://ja.wordpress.org/team/handbook/block-editor/how-to-guides/themes/theme-json/)

------------------------------------------------------

## 4. Astra テーマ の CSS カスタムプロパティ

* [Astra 3.7 - Global Color Palette, Button & Typography Presets](https://wpastra.com/astra-3-7/?_se=dGFrX25hQGljbG91ZC5jb20%3D)
* [Astra Global Color Palette](https://wpastra.com/docs/astra-global-color-palette-settings/)

> Though theme.json enables template editor, since the feature is still in development, we have disabled it until WordPress 5.9 is out.

とある。theme.json はオフにし、テーマ独自に CSS カスタムプロパティを実装している。子テーマで theme.json をオンにし、WordPress コアの CSS カスタムプロパティを使うか、それとも Astra テーマのものを使うか… 

カスタマイザーの 色 > Global Palette で、グローバルに使う色９つを設定できる(3パターン登録できる)。ここでセットしたカラーは、CSS カスタムプロパティとして登録され、Gutenberg エディタ、Elementor、テーマのカラー設定共通でカラーパレットとして反映される。

ページ中でどう反映されるかというと、header タグ内にインラインCSSとして埋め込まれる。(Gutenberg Blocks プラグインを併用している場合は、`<style id="astra-addon-css-inline-css"></style>` として同じく header タグ内にインラインで埋め込まれる。)

```ページ.html
<!-- インラインCSS -->
<style id="astra-theme-css-inline-css">
:root .has-ast-global-color-0-color {
	color: var(--ast-global-color-0);
}
:root .has-ast-global-color-0-background-color {
	background-color: var(--ast-global-color-0);
}
:root .wp-block-button .has-ast-global-color-0-color {
	color: var(--ast-global-color-0);
}
:root .wp-block-button .has-ast-global-color-0-background-color {
	background-color: var(--ast-global-color-0);
}
</style>

<!-- HTML -->
<p class="has-ast-global-color-6-color has-ast-global-color-2-background-color has-text-color has-background">テキスト色、背景色を指定した段落ブロック</p>
```

なので、今まで function.php に書いてた以下の記述が不要となり、、

```functions.php
	add_theme_support( 'editor-color-palette', array(
		// 下のAstraテーマのカラーパレットと揃えること
		array(
			'name'  => esc_attr__( 'primary', 'astra-child' ),
			'slug'  => 'primary',
			'color' => '#96b324',
		),
		// 設定続く
	);
		
/**
 * Astra Theme Color Pallete
 * ※上の Gutenberg カラーパレット設定と揃えること
 */
function astra_child_astra_color_palettes() {
	$color_palettes = array(
		'#96b324',
		'#6c4939',
		'#5A7100',
		'#eb8447',
		'#ff3333',
		'#ffffff',
		'#eeeeee',
		'gray',
		'#313131',
		'#000',
		'#3b312e',
		'#0274be',
	);
	return $color_palettes;
}
add_filter( 'astra_color_palettes', 'astra_child_astra_color_palettes' );
```

同時に、揃えて書いた以下の CSS も不要となる

```style.scss
.has-primary-color {
  color: $primary-color !important;
}
.has-primary-background-color {
  background-color: $primary-color !important;
}
.has-secondary-color {
  color: $secondary-color !important;
}
.has-secondary-background-color {
  background-color: $secondary-color !important;
}
// 続く
```

また、子テーマのプロジェクト直下に theme.json を置くことで、WordPress コアの変数が下にインラインで差し込まれる。順番が後なので、Astra テーマの変数を設定できる。

* [How To Override Astra’s theme.json in Child Theme?](https://wpastra.com/docs/override-astras-theme-json/)

これを使うべきかどうか。。
カラーパレットが９つ固定、テーマが head タグ内に CSS を生成しキャッシュ機能はない、将来を考えるとコアで行っとくほうが良いのではないかと今の所はそう思う。




### CSSグローバル変数を使用してのスタイリング

トップレベルスタイル (body セレクタを使用してエンキューされる)

```theme.json
{
    "version": 1,
    "styles": {
        "color": {
            "text": "var(--wp--preset--color--primary)"
        }
    }
}
```

```style.css (出力)
body { color: var( --wp--preset--color--primary ); }
```

要素スタイル

トップレベルにあれば、要素セレクタが使用されます。ブロック内にあれば、使用されるセレクタは、対応するブロックに追加された形の要素セレクタに

```theme.json
{
    "version": 1,
    "styles": {
        "typography": {
            "fontSize": "var(--wp--preset--font-size--normal)"
        },
        "elements": {
            "h1": {
                "typography": {
                    "fontSize": "var(--wp--preset--font-size--huge)"
                }
            }
        }
    }
}
```

```style.css (出力)
body { font-size: var( --wp--preset--font-size--normal ); }
h1   { font-size: var( --wp--preset--font-size--huge ); }
```

ブロックスタイル

★★TODO: 未

------------------------------------------------------

## 5. いい感じにまとめる (ベストプラクティス)

上記を頭に入れた上で、どのようにすればスッキリ管理できるか考えた

★★TODO: 

## メモ

* [ページ内で使用したブロックのCSSのみを読み込む - WordPress 5.8 | エビスコム - EBISUCOM](https://ebisu.com/note/block-styles-separate-loading/) どうか？