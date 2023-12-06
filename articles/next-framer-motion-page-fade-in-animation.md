---
title: "Next.js x Framer Motion : ページ遷移時にふわっと切り替わるアニメーション"
emoji: "🐸"
type: "tech"
topics: ["nextjs", "framermotion"]
published: true
---

※ SVG を扱う際は注意。公式に「現在 Chrome にはバグがあり、IntersectionObserver が SVG 要素で正しく動作しません。」とある

## 実装の手順

執筆時の環境: next.js 13.5.5, framer-motion 10.16.5

### 1. Framer Motion のインストール

`npm install framer-motion`

### 2. app/template.tsx を作成

以下の内容の template.tsx を作成。これだけで、子要素で遷移時に layout.tsx > template.tsx > その個別ページ と入れ子になり、遷移するごとにふわっと出現するアニメーション (透明度が0→1に変化) が発火する。

:::message
**template.tsx とは**
デフォルトでは存在しないので必要時に作成。このファイルが存在すると、Layout.tsx の内側で各ページや子レイアウトをラップし、その内容をレンダリングする。

layout.tsx と似ているが、ページ遷移時の挙動が異なる。layout.tsx が「ルート間で持続して状態を維持」するのに対し、template.tsx は「ナビゲーション上の各子(子レイアウト、ページ) に対してコンポーネントの新しいインスタンスを作成」する。
:::

```tsx:app/template.tsx
'use client'
import { motion } from 'framer-motion'
import React from 'react'

const variants = {
  hidden: { opacity: 0 },
  enter: { opacity: 1 },
}

export default function Template({ children }: { children: React.ReactNode }) {
  return (
    <motion.div
      className='site-wrapper'
      variants={variants}
      initial='hidden'
      animate='enter'
      transition={{
        type: 'linear',
        duration: 2,
      }}
    >
      {children}
    </motion.div>
  )
}
```


## 参照

* [Scroll animations | Framer for Developers](https://www.framer.com/motion/scroll-animations/)
* [Scroll-Linked Content Reveal Animation](https://www.letsbuildui.dev/articles/scroll-linked-content-reveal-animation/)
* [Next.js 13 + Framer Motion Page Transitions | by Mark Gangel | Oct, 2023 | Stackademic](https://blog.stackademic.com/next-js-13-framer-motion-page-transitions-b2d58658410a)
* [Routing: Pages and Layouts | Next.js](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#templates)
* [Transition | Framer for Developers](https://www.framer.com/motion/transition/)
* [Next.js13.4 + Framer Motion でページ遷移時の ふわっ とした動きを実装してみた。](https://zenn.dev/bloomer/articles/3a814d9f054198)
