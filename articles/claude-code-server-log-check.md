---
title: "Claude Code に「ログ見て」と言うだけでサーバーログを確認させる方法"
emoji: "🐸"
type: "tech"
topics: ["claudecode", "開発環境", "ai", "ai駆動開発"]
published: true
---

開発中、開発用サーバーのコンソールにエラーが出た時に Claude Code に原因を調べ直してほしい。
頻繁に起こるシーンですが、毎回そコピペして貼り付けるのは正直しんどいですよね。

**Unix の `tee` コマンド**と **CLAUDE.md への数行の記述**だけで、「ログ見て」と一言伝えるだけで Claude Code がサーバーログを確認してくれる仕組みが簡単に５分で作れます！

## Claude Code は別ターミナルのログが見えない

まず前提として、Claude Code が操作できるのは**自分が動いているターミナルセッションだけ**です。

例えば、Vite の開発サーバーを別タブや別ペインのターミナルで起動します。Claude Code からはその出力を読む手段がありません(※1)。
（※1: そのセッションでバックグラウンド起動させることもできるし、`!` を付けて実行すればログはコンテキストに含まれますが、別タブで実行しログを眺めたい場面は多いはずです。）
なので、サーバーのログをコピペして会話に貼り付ける手間が発生しますがこれはしたくない…。

解決策はシンプルです。サーバーのログを**ファイルにも書き出す**ようにすれば、Claude Code が `tail` コマンドで読めるようになります！

## tee でログをファイルに書き出す

`tee` は、コマンドの出力を**画面に表示しつつ、同時にファイルにも書き出す** Unix コマンドです。

```bash
# 通常: 出力は画面のみ
npm run dev

# tee: 画面 + ファイル両方に出力
npm run dev 2>&1 | tee /tmp/frontend.log
```

`2>&1` はエラー出力も含めるという意味です。これを付けないと、エラーメッセージがファイルに記録されません。

この 1 行を足すだけで、Claude Code は `/tmp/frontend.log` を `tail` で読める状態になります。

## 実装手順（3 ステップ）

### Step 1: package.json にスクリプトを追加

既存の `dev` コマンドはそのまま残し、`dev:log` を追加します。

```json
{
  "scripts": {
    "dev": "vite",
    "dev:log": "vite 2>&1 | tee /tmp/frontend.log"
  }
}
```

`dev:log` はログをファイルに出力する版、`dev` は従来どおり。ログ出力が不要な場面では `dev` を使えばいいので、既存のワークフローに影響しません。

:::message
Next.js など、`vite` 以外のフレームワークでも同じ要領です。`"next dev 2>&1 | tee /tmp/frontend.log"` のように、既存の dev コマンドに `2>&1 | tee /tmp/ファイル名` を付けるだけです。
:::

**モノレポの場合**は、バックエンドとフロントエンドでファイル名を分けます。

```json
// apps/api/package.json
{
  "scripts": {
    "dev:local": "npx tsx src/local-dev-server.ts",
    "dev:log": "npx tsx src/local-dev-server.ts 2>&1 | tee /tmp/backend.log"
  }
}
```

```json
// apps/web/package.json
{
  "scripts": {
    "dev": "vite",
    "dev:log": "vite 2>&1 | tee /tmp/frontend.log"
  }
}
```

### Step 2: CLAUDE.md にログ確認ルールを記述

汎用的に便利なため、全プロジェクトに適用させたい。なので、グローバル CLAUDE.md（`~/.claude/CLAUDE.md`）に以下を追記します。

```markdown
## ログ確認
- ログの確認を求められたら `/tmp/` 配下のログファイルを `tail -n 100` で確認する
- 代表的なファイル: `/tmp/backend.log`, `/tmp/frontend.log`
- ファイルが存在しない場合は `pnpm dev:log` で起動されていない旨を伝える
- 行数が足りなければ増やす（200, 500 等）
- エラーや警告があれば要約して報告する
```

これで、どのプロジェクトでも同じルールが自動的に適用されます。新しいプロジェクトでも `dev:log` スクリプトを追加するだけで使えるようになります。

### Step 3: 使ってみる

1. `pnpm dev:log` でサーバーを起動する（画面表示は従来と同じ）
2. Claude Code に「ログ見て」「エラー出てない？」などと伝える
3. Claude Code が `tail -n 100 /tmp/frontend.log` を実行し、ログを確認・要約してくれる

これで完了です。

## ログファイルの肥大化が心配？

`tee` はログを追記し続けるので、「ファイルが大きくなるのでは？」という懸念があるかもしれません。しかし、サーバーを再起動するたびに新しい teeプロセスが始まるためログファイルはリセットされるので、その心配は不要です。

## 応用 — モノレポでの使い分け

モノレポでバックエンドとフロントエンドが別々に動いている場合は、ログファイル名を分けます。

```json
// apps/api/package.json
{
  "scripts": {
    "dev:local": "npx tsx src/local-dev-server.ts",
    "dev:log": "npx tsx src/local-dev-server.ts 2>&1 | tee /tmp/backend.log"
  }
}
```

```json
// apps/web/package.json
{
  "scripts": {
    "dev": "vite",
    "dev:log": "vite 2>&1 | tee /tmp/frontend.log"
  }
}
```

プロジェクト固有の情報（どのスクリプトがどのログファイルを出すか）は、プロジェクトルートの `CLAUDE.md` に書いておくと、チームメンバーにも共有できます。

```markdown
### ログ確認
- バックエンドが `pnpm dev:log`（`apps/api`）で起動されている場合、`/tmp/backend.log` に出力される
- フロントエンドが `pnpm dev:log`（`apps/web`）で起動されている場合、`/tmp/frontend.log` に出力される
```

グローバル CLAUDE.md の汎用ルールと、プロジェクト CLAUDE.md の固有情報を分離することで、設定の再利用性が高まります。

## まとめ

やることは 3 つだけです。

1. `package.json` に `dev:log` スクリプトを追加（`2>&1 | tee /tmp/ファイル名`）
2. グローバル CLAUDE.md（`~/.claude/CLAUDE.md`）にログ確認ルールを記述
3. `dev:log` で起動して「ログ見て」と言う
