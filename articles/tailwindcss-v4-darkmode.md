---
title: "Tailwind CSS v4 のネイティブダークモードの概要と v3 からの移行手順"
emoji: "🐸"
type: "tech"
topics: ["tailwindcss", "nextjs"]
published: true
---

Tailwind CSS v4 のネイティブダークモード機能を活用して、Next.jsアプリケーションにおけるダークモードの実装方法のベストプラクティスが変わります。

この記事では、Tailwind CSS v3 の `next-themes` ライブラリでの実装から v4 のネイティブダークモード機能への移行プロセスなどを解説します。

## 新しい実装方法を採用するメリット

1. コードの簡素化でメンテナンスしやすく
2. `next-themes` ライブラリへの依存が不要に(バンドルサイズの削減、アップデートリスクの軽減)

### CSSの実装がシンプルに

Tailwind CSS v3 では `dark:` プレフィックスが必要でした。

```tsx
<div className="bg-white dark:bg-slate-800">
  <p className="text-black dark:text-white">テキスト</p>
</div>
```

Tailwind CSS v4 では、CSS変数を活用することでテーマ管理が容易になります。
カスタムプロパティの活用で、色やテーマ値をよりシステマティックに管理できるようになりました。
ダークモードのスタイルの定義は `@custom-variant` を使用します。

```tsx
<div className="bg-background">
  <p className="text-foreground">テキスト</p>
</div>
```

```css
/* globals.css */

/* ライトモード用のカスタム変数 */
:root {
  --background: 0 0% 100%;
  --foreground: 0 0% 0%;
}

@custom-variant dark (&:where(.dark, .dark *)) {
  /* ダークモード用のカスタム変数 */
  --background: 240 10% 3.9%;
  --foreground: 0 0% 98%;
}
```

## Tailwind CSS v3 から v4 のネイティブダークモードへの移行手順

※サンプルコードはわかりやすさのためシンプルに加工しています。

## メモ: Tailwind CSS v3 までの実装

v3 の実装では、`next-themes` ライブラリを使用してダークモード機能(ライト/ダークモードの切替)を実現していました。ユーザー設定の localStorage 保存、フラッシュ防止機能も実装する必要があります(これはv4でも同じ)。

```tsx
<ThemeProvider
  attribute="class"
  defaultTheme="light"
  enableSystem
  disableTransitionOnChange
>
  {children}
</ThemeProvider>
```

### 1. ThemeProvider コンポーネントの書き換え

```tsx
"use client";

import { useEffect } from "react";

export function ThemeProvider({
  children,
  defaultTheme,
}) {
  useEffect(() => {
    // localStorageからテーマを取得
    const storedTheme = localStorage.getItem('theme');
    const currentTheme = storedTheme || defaultTheme;

    // テーマに応じてdarkクラスの付け外し
    document.documentElement.classList.toggle("dark", currentTheme === "dark");
  }, [defaultTheme]);

  return <>{children}</>;
}
```

### 2. テーマ切替コンポーネント (モード切り替えボタン) の実装

```tsx
"use client";

import { useState, useEffect } from "react";

export default function ThemeSwitcher() {
  const [theme, setTheme] = useState("light");
  
  useEffect(() => {
    // 初期テーマの取得
    const storedTheme = localStorage.getItem('theme');
    const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
    
    // 保存されたテーマがあればそれを使用、なければシステム設定を確認
    if (storedTheme) {
      setTheme(storedTheme);
    } else if (prefersDark) {
      setTheme("dark");
    }
  }, []);

  const toggleTheme = () => {
    const newTheme = theme === "light" ? "dark" : "light";
    // テーマの更新
    setTheme(newTheme);
    localStorage.setItem('theme', newTheme);
    document.documentElement.classList.toggle("dark", newTheme === "dark");
  };

  // システムテーマの変更を監視（オプション）
  useEffect(() => {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    const handleChange = (e) => {
      if (!localStorage.getItem('theme')) {
        // ユーザーが明示的にテーマを設定していない場合のみ適用
        const newTheme = e.matches ? "dark" : "light";
        setTheme(newTheme);
        document.documentElement.classList.toggle("dark", e.matches);
      }
    };

    mediaQuery.addEventListener('change', handleChange);
    return () => mediaQuery.removeEventListener('change', handleChange);
  }, []);

  return (
    <button onClick={toggleTheme}>
      {theme === "light" ? "🌙" : "☀️"}
    </button>
  );
}
```

### 3. 問題解決：ページ読み込み時のフラッシュ(チラつき)防止

ページ読み込み時にテーマのフラッシュ（チラつき）が発生する問題を解決するため、早期のテーマ初期化スクリプトを `layout.tsx` に追加し、React コンポーネントのハイドレーション前(マウント前)にテーマを適用させます。
このアプローチは、他の機能の実装にも応用可能です。

```tsx
<head>
  <script
    dangerouslySetInnerHTML={{
      __html: `
        (function() {
          try {
            const theme = localStorage.getItem('theme');
            if (theme === 'dark') {
              document.documentElement.classList.add('dark');
            } else if (!theme && window.matchMedia('(prefers-color-scheme: dark)').matches) {
              document.documentElement.classList.add('dark');
            } else {
              document.documentElement.classList.remove('dark');
            }
          } catch (e) {
            console.error('テーマ初期化エラー:', e);
          }
        })();
      `,
    }}
  />
</head>
```

### 4. next-themes パッケージの削除

不要になったパッケージを削除します。
`npm uninstall next-themes`

## まとめ

Tailwind CSS v4 のネイティブダークモード機能で、外部ライブラリに依存せず、シンプルにダークモード機能を実装できるようになります。シンプルになるので、古いブラウザでの動作も考慮しなければならないプロジェクトでないなら、ぜひ導入しましょう。

## 参考リソース

- [Tailwind CSS公式ドキュメント - ダークモード](https://tailwindcss.com/docs/dark-mode)
- [Tailwind CSS v4 アップグレードガイド](https://tailwindcss.com/docs/upgrade-guide)
