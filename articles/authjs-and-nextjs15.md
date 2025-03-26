---
title: "Next.js 15 に Auth.js (Next Auth v5) 認証を導入"
emoji: "🐸"
type: "tech"
topics: ["nextjs", "authjs", "next-auth"]
published: true
---

[Next.js 15](https://authjs.dev)と [Auth.js](https://authjs.dev)（旧Next Auth v5）を使って、認証機能を実装する方法を解説します。
Turbopackを使用している場合の注意点も含めて説明します。

## 目次

1. [Auth.jsとは](#1-authjsとは)
2. [必要なパッケージのインストール](#2-必要なパッケージのインストール)
3. [Auth.js設定ファイルの作成](#3-authjs設定ファイルの作成)
4. [API Routeの設定](#4-api-routeの設定)
5. [認証プロバイダーの設定](#5-認証プロバイダーの設定)
6. [認証UIコンポーネントの作成](#6-認証uiコンポーネントの作成)
7. [閲覧に認証が必要なページの作成](#7-閲覧に認証が必要なページの作成)
8. [環境変数の設定](#8-環境変数の設定)
9. [動作確認](#9-動作確認)
10. [Turbopackでの注意点](#10-turbopackでの注意点)

## 1. Auth.jsとは

Auth.js（旧Next Auth）は、Next.jsアプリケーションに認証機能を簡単に追加できるライブラリです。
バージョン5からは「Auth.js」という名称に変更され、Next.js以外のフレームワークでも使えるように改善されました。
主な特徴は以下の通りです。

- 多くの認証プロバイダー（GitHub, Google, X, Line, etc.）をサポート
- セッション管理の簡略化
- JWT（JSON Web Token）による認証
- カスタマイズ可能なコールバック
- TypeScriptサポート

### Auth.js を選んだ理由

Clerk も人気があり良さそうなので迷いましたが、以下の理由でAuth.jsを選びました。

- **自己ホスティング**: Clerk は商用SaaSサービス、Auth.js は自己ホスティング型でオープンソース
- **対応認証プロバイダー**: 最新版でLINE認証に対応
- **Edge Runtime**: 最新版でEdge Runtimeに対応
- **料金**: Auth.js は無料、Clerk は無料枠を超えると課金が必要

### Edge Runtime（Cloudflare Workers 等）での動作

Auth.js v5はEdge Runtime対応です。
ただし、いくつか注意点があります。

- **コア機能**: 基本的な認証機能はEdge Runtimeで問題なく動作
- **依存ライブラリ**: 使用する外部ライブラリもEdge対応である必要あり
- **セッション管理**: JWT方式のセッションが推奨（データベースセッションはアダプターの対応状況に依存）

シンプルな認証機能であればEdgeで動作しますが、データベース連携など複雑な実装では互換性の確認が必要です。

- [Edge Compatibility - Auth.js](https://authjs.dev/guides/edge-compatibility)
- [Cloudflare PagesでNextAuth（Edge Runtime） - Zenn](https://zenn.dev/niwatoro/articles/b87071718ac836)
- [NextjsのランタイムとCloudflare workersについて - Zenn](https://zenn.dev/takiko/scraps/99d7ec3bda94ac)

---

## 2. パッケージのインストール

`next-auth@beta`をインストール (2025-03-26時点ではベータ版)。

```bash
npm install next-auth@beta
```

## 3. Auth.js設定ファイルの作成

今回は、GitHubによるソーシャル認証を設定します。  
プロジェクトのルートに`src/auth.ts`ファイルを作成し、Auth.jsの設定を記述します。

```typescript
import NextAuth from "next-auth";
import GitHub from "next-auth/providers/github";
// 他の認証プロバイダーをインポートすることもできる

// Auth.js v5の正しい設定
export const { handlers, signIn, signOut, auth } = NextAuth({
  debug: process.env.NODE_ENV === "development", // 開発環境ではデバッグ情報を表示
  providers: [
    GitHub({
      clientId: process.env.AUTH_GITHUB_ID ?? "",
      clientSecret: process.env.AUTH_GITHUB_SECRET ?? "",
    }),
  ],
  // カスタムページを実装する場合は設定する
  // pages: {
  //   signIn: "/auth/signin",
  //   error: "/auth/error",
  //   signOut: "/auth/signout",
  // },
  callbacks: {
    // 必要に応じてコールバックを追加
    async session({ session, token }) {
      // JWT からセッションに情報を渡す
      return session;
    },
    async jwt({ token, user }) {
      // ユーザーがログインするときに JWT に情報を追加
      if (user) {
        token.id = user.id;
      }
      return token;
    },
  },
});
```

## 4. API Routeの設定

`src/app/api/auth/[...nextauth]/route.ts`ファイルを作成します。

```typescript
// Auth.js v5 の API ルート実装
import { handlers } from "@/auth";

// Turbopack 対応のエクスポート方法
export const GET = handlers.GET;
export const POST = handlers.POST;
```

## 5. 認証プロバイダーの設定

全てのコンポーネントで `useSession` フックを使用できるように、認証プロバイダーを設定。
まず、`src/components/auth/auth-provider.tsx`ファイルを作成します。

```tsx
"use client";

import { SessionProvider } from "next-auth/react";

export function AuthProvider({ children }: { children: React.ReactNode }) {
  return <SessionProvider>{children}</SessionProvider>;
}
```

次に、このプロバイダーをアプリケーションのルートレイアウトに追加（`src/app/layout.tsx`）。

```tsx
import { AuthProvider } from "@/components/auth/auth-provider";

// ...その他のインポート...

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="ja" suppressHydrationWarning>
      <body>
        <ThemeProvider
          attribute="class"
          defaultTheme={THEME.DEFAULT}
          enableSystem
          disableTransitionOnChange
        >
          <AuthProvider>
            {/* その他のコンポーネント */}
            <div className="flex flex-col min-h-screen">
              <Header />
              <main className="flex-grow">
                {children}
              </main>
              <Footer />
            </div>
          </AuthProvider>
        </ThemeProvider>
      </body>
    </html>
  );
}
```

## 6. 認証UIコンポーネントの作成

サインイン/サインアウト のボタンコンポーネントを作成します。  
`src/components/ui/auth-buttons.tsx`。

```tsx
"use client";

import { useSession, signIn, signOut } from "next-auth/react";
import { Button } from "@/components/ui/button";

export function AuthButtons() {
  const { data: session, status } = useSession();
  
  if (status === "loading") {
    return <div>Loading...</div>;
  }
  
  if (session) {
    return (
      <div className="flex items-center gap-4">
        <p>ようこそ、{session.user?.name ?? "ゲスト"}さん</p>
        <Button 
          variant="outline" 
          type="button"
          onClick={() => signOut()}
        >
          サインアウト
        </Button>
      </div>
    );
  }
  
  return (
    <Button 
      type="button"
      onClick={() => signIn("github")}
    >
      GitHub でサインイン
    </Button>
  );
}
```

このコンポーネントをページに追加して使用します（例：`src/app/page.tsx`）。

```tsx
import { AuthButtons } from "@/components/ui/auth-buttons";

export default function Home() {
  return (
    <div className="container py-12">
      <h1 className="text-4xl font-bold">Next.js + Auth.js デモ</h1>
      <div className="mt-8">
        <AuthButtons />
      </div>
    </div>
  );
}
```

## 7. 閲覧に認証が必要なページの作成

`src/app/dashboard/page.tsx`。

```tsx
import { auth } from "@/auth";
import { redirect } from "next/navigation";

export default async function DashboardPage() {
  const session = await auth();

  // ユーザーがログインしていない場合はリダイレクト
  if (!session) {
    redirect("/auth/signin");
  }

  return (
    <div>
      <h1>ダッシュボード</h1>
      <p>こんにちは、{session.user?.name}さん！</p>
      <p>
        これは保護されたページです。ログインユーザーだけが見ることができます。
      </p>
    </div>
  );
}
```

## 8. 環境変数の設定

`.env.local`に必要な環境変数を設定 (まだ無ければ作成)。

```.env.local
# Auth.js (AUTH_URL は自動検出)
AUTH_SECRET="任意の長い文字列（セッション暗号化用）"
AUTH_GITHUB_ID="GitHubで取得したクライアントID"
AUTH_GITHUB_SECRET="GitHubで取得したクライアントシークレット"
```

GitHubの認証情報は、GitHubの[Developer Settings](https://github.com/settings/developers)から新しいOAuthアプリケーションを作成して取得できます。

## 9. 動作確認

アプリケーションを起動して、認証機能が正しく動作するか確認します。

```bash
npm run dev
```

認証が必要なページ（`/dashboard`）にアクセスすると、サインインページにリダイレクトされます。  
サインインボタンをクリックすると、GitHubの認証ページにリダイレクトされ、認証後にアプリケーションに戻ります。  
認証した状態で`/dashboard`にアクセスすると、ダッシュボードページが表示されます。

## 9. Turbopackでの注意点

Next.js 15でTurbopackを使用する場合、いくつかの注意点があります。

1. **API Routeの実装方法**
   - 分割代入でのエクスポートではなく、個別のプロパティとしてエクスポートします

   ```typescript
   // 分割代入でのエクスポート（問題がある場合がある）
   export const { GET, POST } = handlers;
   
   // 個別プロパティのエクスポート（推奨）
   export const GET = handlers.GET;
   export const POST = handlers.POST;
   ```

2. **カスタムページの設定**
   - カスタムページを実装する前に`pages`オプションを設定すると、404エラーが発生します
   - 実際にカスタムページを実装するまでは、この設定をコメントアウトしておきましょう

3. **環境変数のデフォルト値**
   - 環境変数にデフォルト値を設定して、未設定時のエラーを回避します

   ```typescript
   GitHub({
     clientId: process.env.AUTH_GITHUB_ID ?? "",
     clientSecret: process.env.AUTH_GITHUB_SECRET ?? "",
   }),
   ```

以上の点に注意することで、Next.js 15とAuth.js（Next Auth v5）を使った認証機能をスムーズに実装できます。

## まとめ

Next.js 15 のアプリケーションに Auth.js（Next Auth v5）で認証機能を実装する方法を解説しました。
複雑な認証ロジックを簡単に実装でき、セキュアなアプリケーション開発が可能になります。
