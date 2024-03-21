---
title: "React: リスト要素と複合コンポーネント"
emoji: "🐸"
type: "tech"
topics: ["react", "frontend"]
published: true
---

## 複合コンポーネントとは

複合コンポーネントは、小さい単純なコンポーネントを組み合わせ、ひとつのコンポーネントを作る技術。例えば、ボタン/入力フォーム/コンポーネントを組み合わせて商品の注文フォームを作ったり、タイトル/本文/フッターコンポーネントを組み合わせてカードコンポーネントを作ったり。

今回は、単純化したページ内アンカーリンクのリストコンポーネントををサンプルとして説明する。

## 単一コンポーネントの場合

TocLink コンポーネント側でリンク一式の配列データを受け取り、各子要素を map 関数のループ処理で組み立てる

```tsx:components/TocLink.tsx
function TocLink(data: { id: string; title?: string }[]) {
  return (
    <ul className="XXX">
      {data.map(({ id, title }) => {
        return (
          <li key={id}>
            <a className="YYY" href={`#${id}`}>
              {title ? title : id}
            </a>
          </li>
        )
      })}
    </ul>
  )
}

export default TocLink
```

使用するページでリンクのデータ (オブジェクトの配列) 一式を用意し、TocLink コンポーネントに props として渡す。

```tsx:access/page.tsx
import TocLink from '@/components/TocLink'

export default function Page() {
  const TocData = [
      {id: 'google-map', title: 'Google マップ'},
      {id: 'access-by-train', title: '電車でのアクセス'},
      {id: 'access-by-car', title: '車でのアクセス'},
  ]

  return (
      <TocLink data={TocData} />
  )
}
```

出力 HTML

```html
<ul class="XXX">
    <li><a class="YYY" href="#google-map">Google マップ</a></li>
    <li><a class="YYY" href="#access-by-train">電車でのアクセス</a></li>
    <li><a class="YYY" href="#access-by-car">車でのアクセス</a></li>
</ul>
```

* データと使用箇所を別の場所で管理するので、コード量が増えるとわかりづらくなる。
* 同じデータを複数箇所で使う場合や外部データを読み込む場合はシンプルに実装できる。



## 複合コンポーネントにした場合

`ul` 要素と `li` 要素用のコンポーネントを別々に用意し組み合わせる。出力 HTML は単一コンポーネントのサンプルのものと同じ。

```tsx:components/TocLink.tsx
import React from 'react'

function TocLink({ children }: { children: React.ReactNode }) {
  return (
    <ul className="XXX">
      {children}
    </ul>
  )
}

export function TocLinkItem({
  id,
  children,
}: {
  id: string
  children: string
}) {
  return (
    <li key={id}>
      <a className='YYY' href={`#${id}`}>
        {children}
      </a>
    </li>
  )
}

export default TocLink
```

使用ページでは、各リンクのデータを子要素の props として渡す。
リスト項目は children としてタグ内に書くようにすると直感的。
使用箇所に直接データを書くので見通しが良い。

```tsx:access/page.tsx
import TocLink, { TocLinkItem } from '@/components/TocLink'

export default function Page() {
  return (
      <TocLink>
        <TocLinkItem id='google-map'>Google マップ</TocLinkItem>
        <TocLinkItem id='access-by-train'>電車でのアクセス</TocLinkItem>
        <TocLinkItem id='access-by-car'>車でのアクセス</TocLinkItem>
      </TocLink>
  )
}
```

## まとめ

* リスト要素のように親子で構成されたタグで子要素をループ処理をする場合、複合コンポーネントとすると見通しがよく扱いやすい。
* 同じデータを複数箇所で使う場合や外部データを読み込む場合は、単一コンポーネントの方がシンプルに実装できる。
