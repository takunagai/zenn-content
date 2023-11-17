---
title: "React: コンポーネントの呼び出し元でクラスや属性を付加する"
emoji: "🐸"
type: "tech"
topics: ["react", "tailwindcss", "css"]
published: true
---

コンポーネントに、その場で必要なスタイルを付けたいというシーンがある。
例えば、「ここでは上余白を入れたい」とか。

そういった場合、コンポーネント側で `className` prop を受け取るようにしておき、それを [clsx](https://www.npmjs.com/package/clsx) ライブラリでコンポーネントが持つクラスと統合させてやれば良い。コンポーネントの CSS を CSS Modules で設定している場合でも、Tailwind 等を併用している場合でも問題なくいける。

TailwindCSS なら [tailwind-merge](https://www.npmjs.com/package/tailwind-merge) ライブラリで競合解決できるので、clsx と tailwind-merge の処理を組み合わせたユーティリティ関数を用意しておけば便利。([chadcn/ui](https://ui.shadcn.com/) は、コンポーネント生成時にその関数を自動で生成してくれるようになっており、この記事のサンプルはそれをそのまま流用したもの。)

また、個別に要素に属性など (`data-XXX` とか) を付けたい場合は、コンポーネントで明示的に受け取る prop 以外の残り全てのものをスプレッド構文で受け取るようにしておき、返す HTML 要素にセットしてやれば、呼び出し元でセットした属性値がそのまま引き継がれてセットされる。

## 実装例

単純な Card コンポーネントを例とする。

```txt
// 構成
プロジェクトルート
  ├ app
  │ ├ pagename
  │ │ ├ page.tsx
  │
  ├ components
  │ ├ ui
  │   └ Card
  │     ├ index.tsx
  │     ├ Card.tsx
  │     └ Card.module.css
  │
  ├ utils.ts

```

```components/ui/Card/Card.tsx
// components/ui/Card/Card.tsx
import { cn } from '@/utils'
import React from 'react'
import styles from './Card.module.css'

function Card({
  children,
  className,
  ...delegated
}: {
  children: React.ReactNode
  className?: string
}) {
  return (
    <div className={cn(styles.wrapper, className)} {...delegated}>
      {children}
    </div>
  )
}

export default Card
```

```components/ui/Card/Card.module.css
// components/ui/Card/Card.module.css
/* Card の基本スタイル */
```

```utils.ts
// utils.ts
// 抽象的なタスクを実行する汎用関数
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}
```

```app/pagename/page.tsx
// app/pagename/page.tsx
import Card from '@/components/ui/Card'

export default function Page() {
  return (
    <Card
      className='p-4 md:p-6 lg:p-8'
      data-id="10784"
    >
      <h3>見出しです</h3>
      <p>テキストです。</p>
    </Card>
  )
}
```
