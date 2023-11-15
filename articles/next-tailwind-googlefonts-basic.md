---
title: "Next.js x TailwindCSS: Google Fonts (Noto Sans JP) の使い方"
emoji: "🐸"
type: "tech"
topics: ["nextjs", "tailwindcss", "frontend"]
published: false
---

ややこしかったし、ググっても明快な情報を探せなかったのでメモしとく。  
今回は Noto Sans JP を設定したが、他のフォントでも同様の方法でいけるはず。  
環境は、Next.js 13.5.5 (App Router)

## 設定の手順

今回は、ページ全体 (html 要素) に Noto Sans JP を適用する手順。  
編集するのは layout.tsx、tailwind.config.ts の２ファイル。  
`font-sans` クラスで Noto Sans JP が適用されるようにする。  

※ この記事後半の "出力された HTML" の項を先に見た方がわかりやすいかも。

### 1. layout.tsx

まずは、layout.tsx で Web フォントを読み込み設定する(インスタンス生成)。  
ドキュメントを見ると様々なオプションがあるが、ほぼデフォルトで OK。

:::message
Noto フォントは "バリアブルフォント" なので、ウェイト指定は不要。むしろ、"weight" を指定すると、[CSS サイズが増える](https://zenn.dev/k_kind/articles/next-font-weight)らしい。  

当該フォントがバリアブルフォントかどうかを知るには、[Variable fonts 一覧](https://fonts.google.com/variablefonts) にそのフォント名があるか確認するか、そのフォントの個別ページ)の "Type tester" (例: [Noto Sans JP]([Noto Sans Japanese - Google Fonts](https://fonts.google.com/noto/specimen/Noto+Sans+JP/tester)) の Weight スライダー でウェイトが 1 ずつ変更できるかを確認すれば良い。
:::

以下のサンプルは、全体 (html 要素) に Noto Sans JP を適用するもの。

```tsx
// app/layout.tsx
import { Noto_Sans_JP } from 'next/font/google'
import clsx from 'clsx' // 今回の実装に必須というわけではないが便利

const notoSansJP = Noto_Sans_JP({
  subsets: ['latin'],
  variable: '--font-noto-sans-jp',
  // weight: 'variable', // default なので不要。バリアブルフォントでなければ必要
  // display: 'swap', // default なので不要
  // preload: true, // default なので不要
  // adjustFontFallback: true, // next/font/google で default なので不要
  // fallback: ['system-ui', 'arial'], // local font fallback なので不要
})

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang='ja' className={clsx(notoSansJP.variable, 'font-sans')}>
      {/* 略 */}
    </html>
  )
}
````

### 2. tailwind.config.ts

theme.fontFamily.sans の値に layout.tsx で設定した variable をセットする。
これで、`font-sans` クラスで Noto Sans JP が適用される。

```ts
// tailwind.config.ts

// 略
  theme: {
    fontFamily: {
      sans: ['var(--font-noto-sans-jp)'], // font-sans クラスに適用
    },
```

## 結果 (出力された HTML)

html 要素にセットした `notoSansJP.variable` が、
`__variable_8f86ba` のようなクラスとして付加されており…

```html
<html lang="ja" class="__variable_8f86ba font-sans">
```

そのクラスには、以下のようにフォント用の CSS 変数がセットされている

```css
.__variable_8f86ba {
  --font-noto-sans-jp:
      '__Noto_Sans_JP_8f86ba',
      '__Noto_Sans_JP_Fallback_8f86ba';
}
```

Tailwind の設定で font-sans クラスに `var(--font-noto-sans-jp)` を充てているので、結果、'__Noto_Sans_JP_8f86ba' がフォントファミリーとして適用されるという仕組み。

## メモ

* 同じフォントを複数の場所で使用する場合、[フォント定義ファイル](https://nextjs.org/docs/pages/api-reference/components/font#using-a-font-definitions-file)を使うと良い。このファイルには複数の Google フォントやローカルフォントをセットできる。(※本文や見出しなど、どのページでも使うようなフォントは app/layout.tsx に適用するだけでよいので不要だと思う)
* 例えば、ページ中で見出しに別の Google Font を使うと言った場合は、html 要素にその見出し用フォントの variable を設定する。これでフォント用の CSS 変数が html 要素内、つまり全体に適用される。

## あとがき

結局、軽さ重視で Google フォントを使うのはやめることに…

## 参照

* [Components: Font | Next.js](https://nextjs.org/docs/pages/api-reference/components/font#declarations)