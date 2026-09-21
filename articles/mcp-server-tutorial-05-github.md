---
title: "【MCPのトリセツ #5】GitHub MCPサーバー： AIでリポジトリを管理"
emoji: "🐸"
type: "tech"
topics: ["mcp", "github", "claude", "claudecode", "codex"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズです。今回は GitHub 公式の MCP サーバーを取り上げます。リポジトリの検索、Issue の整理、プルリクエストの作成を、AI との会話から行えるようになります。

:::message
**更新日: 2026-09-22**（初版: 2025-03-08）

旧版で紹介していた `@modelcontextprotocol/server-github` は開発が終了しアーカイブされたため、後継である GitHub 公式の `github/github-mcp-server` の手順に差し替えています。
:::

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. **GitHub MCPサーバー： AIでリポジトリを管理（この記事）**
6. [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. [Slack MCPサーバー：チームコミュニケーションを強化](./mcp-server-tutorial-07-slack)
8. [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
9. [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)
11. [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
12. [Blender MCP Server: 会話で Blender を操作し3Dモデルを作成](./mcp-server-tutorial-12-blender)
13. [Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行](./mcp-server-tutorial-13-perplexity)
14. [国土交通省がMCPサーバーを公開：AI時代のオープンデータ活用45選](./mcp-server-tutorial-14-milt-data)

資料: [ウェブ情報を取得するMCPの比較 (Fetch、Firecrawl、Markdownify、Perplexity)](./mcp-server-tutorial-reference-web-mcp)

---

## GitHub MCP サーバーとは

[github/github-mcp-server](https://github.com/github/github-mcp-server) は、GitHub 自身が開発・運用している MCP サーバーです。使い方は 2 通りあります。

| 形態 | 概要 | 認証 |
|---|---|---|
| リモート版 | GitHub がホストする `https://api.githubcopilot.com/mcp/` に接続する | 個人アクセストークン（PAT） |
| ローカル版 | Docker イメージ `ghcr.io/github/github-mcp-server` を手元で動かす | ブラウザでの OAuth ログイン、または PAT |

Claude Code と Codex はリモート版が手早く、Claude Desktop はローカル版を使います（GitHub の公式ガイドによると、Claude Desktop からのリモート接続は現時点で未対応です）。

機能は「toolsets」という単位でまとめられていて、何も指定しない場合は `context`・`repos`・`issues`・`pull_requests`・`users` が有効になります。GitHub Actions、コードスキャン、Dependabot、Discussions、Projects などは、必要になったときに追加で有効化する設計です。

## 個人アクセストークンを用意する

リモート版を使う場合は PAT が要ります。GitHub は classic よりも fine-grained トークンを推奨しているので、こちらで作成します。

1. GitHub の右上のアイコンから「Settings」を開く
2. 左メニュー最下部の「Developer settings」を開く
3. 「Personal access tokens」の「Fine-grained tokens」から「Generate new token」を押す
4. 名前と有効期限を決める（30 日程度が目安）
5. 「Repository access」で「Only select repositories」を選び、AI に触らせるリポジトリだけを指定する
6. 「Permissions」で必要な権限だけを付ける（読み書きするなら Contents・Issues・Pull requests を「Read and write」にする）
7. 「Generate token」を押し、表示されたトークンを控える（再表示されません）

トークンは AI とのチャット欄に貼らないでください。漏れた可能性があるときは、すぐに同じ画面から失効させます。

## セットアップ手順

### Claude Code

リモート版を登録します。`YOUR_GITHUB_PAT` を作成したトークンに置き換えてください。

```bash
claude mcp add-json github '{"type":"http","url":"https://api.githubcopilot.com/mcp","headers":{"Authorization":"Bearer YOUR_GITHUB_PAT"}}'
```

登録後、`claude mcp list` で `github` が接続済みになっていれば完了です。トークンは Claude Code の設定に平文で保存されるため、権限を絞った短命のトークンを使うのが前提です。

### Codex

Codex はトークンを環境変数から読む書き方ができるので、設定ファイルにトークン本体が残りません。

```bash
codex mcp add github --url https://api.githubcopilot.com/mcp/ \
  --bearer-token-env-var GITHUB_PAT_TOKEN
```

`~/.codex/config.toml` には次のように保存されます。環境変数 `GITHUB_PAT_TOKEN` は、シェルの設定などで別途用意します。

```toml
[mcp_servers.github]
url = "https://api.githubcopilot.com/mcp/"
bearer_token_env_var = "GITHUB_PAT_TOKEN"
```

### Claude Desktop

Docker でローカル版を動かします。[Docker Desktop](https://www.docker.com/products/docker-desktop/) を起動した状態で、設定ファイル（開き方は[シリーズ #1](./mcp-server-tutorial-01-install) を参照）に次を追加します。

```json
{
  "mcpServers": {
    "github": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-p",
        "127.0.0.1:8085:8085",
        "-e",
        "GITHUB_OAUTH_CALLBACK_PORT",
        "ghcr.io/github/github-mcp-server"
      ],
      "env": {
        "GITHUB_OAUTH_CALLBACK_PORT": "8085"
      }
    }
  }
}
```

この設定はブラウザでの OAuth ログインを使うため、トークンを設定ファイルに書かずに済みます。Claude Desktop を再起動し、最初にツールを使うときにブラウザで GitHub へのアクセスを許可します。

## 権限を絞る設定

読み取りだけで足りる用途なら、読み取り専用にしておくと事故が減ります。リモート版はリクエストヘッダーで制御します。

```json
{
  "type": "http",
  "url": "https://api.githubcopilot.com/mcp/",
  "headers": {
    "Authorization": "Bearer YOUR_GITHUB_PAT",
    "X-MCP-Toolsets": "repos,issues",
    "X-MCP-Readonly": "true"
  }
}
```

| 設定 | リモート版 | ローカル版 |
|---|---|---|
| 使う機能を絞る | `X-MCP-Toolsets` ヘッダー | `--toolsets` または環境変数 `GITHUB_TOOLSETS` |
| 読み取り専用 | `X-MCP-Readonly: true` | `--read-only` |
| ロックダウン | `X-MCP-Lockdown: true` | ─ |

ロックダウンは、公開リポジトリで push 権限のないユーザーが作成した Issue の内容を AI に渡さないようにするモードです。後述のプロンプトインジェクション対策になります。

## 使用例（プロンプト）

既定の toolsets で動く範囲の例です。リポジトリ名は `owner/repo` の形で伝えると取り違えが起きにくくなります。

### リポジトリの基本操作

```text
「my-portfolio」という名前の新しいリポジトリを作成して。説明は「個人ポートフォリオサイト」、プライベートリポジトリで、README も初期化して
```

```text
myusername/my-portfolio に「feature/user-authentication」ブランチを main から作成して
```

```text
src/components/Login.js を作成し、React のログインコンポーネントを実装して。コミットメッセージは「ログインフォームコンポーネントの追加」で
```

```text
「feature/user-authentication」から「main」へのプルリクエストを作成して。タイトルは「ユーザー認証機能の実装」、説明には機能の概要とテスト方法を含めて
```

### 情報の検索と分析

```text
「react state management」に関する、最近更新された人気のリポジトリを 5 つ探して、それぞれの特徴を簡潔に説明して
```

```text
facebook/react で最近クローズされたバグ修正の Issue を 5 つ見つけて、それぞれどんな問題が解決されたのか説明して
```

```text
myusername/my-project の未解決 Issue をリストアップして、優先度ごとに分類して。比較的早く片付きそうなものを 3 つ選んで
```

### 開発支援

```text
「bugfix/payment-processing」ブランチと「main」ブランチの差分を分析し、どんな問題が修正されたのか説明して
```

```text
このバグについて Issue を作成して：「ログイン後にユーザープロフィールが正しく表示されない。Firefox で再現する」。ラベルは bug と frontend を付けて
```

```text
myusername/my-app の v1.5.0 から現在までのコミットを調べて、v2.0.0 のリリースノートの下書きを作って。新機能、変更点、バグ修正に分けて
```

```text
myusername/backend-api と myusername/frontend-client を読み比べて、API インターフェースの不整合がないか確認して
```

手元のコードを編集してコミット・プッシュする作業は、Claude Code や Codex なら `git` と `gh` コマンドを直接使う方が速く確実です。GitHub MCP が向くのは、「手元にクローンしていないリポジトリを調べる」「Issue とプルリクエストを整理する」といった用途です。MCP と CLI・スキルの選び方は[シリーズ #1](./mcp-server-tutorial-01-install) にまとめました。

## 安全に使うための注意点

GitHub MCP サーバーは、公開の Issue やプルリクエストに書かれた文章を AI に読み込ませます。2025 年 5 月には、悪意のある Issue を読ませることで、同じトークンでアクセスできるプライベートリポジトリの内容を公開の場へ書き出させる手口が Invariant Labs から報告されました。サーバーの不具合ではなく、「外部の文章を読む AI に広い権限を渡す」構成そのものに由来するリスクです。

対策は権限を狭めることに尽きます。

- fine-grained トークンで、対象リポジトリと権限を必要な範囲に限定します
- 調べ物だけなら読み取り専用にします。公開リポジトリを扱うならロックダウンも有効にします
- 書き込み系ツールの実行は自動承認にせず、内容を確認してから許可します
- トークンには有効期限を付け、使わなくなったら失効させます

## 他の MCP サーバーとの組み合わせ

[Filesystem MCP](./mcp-server-tutorial-02-filesystem) と組み合わせると、GitHub 上の Issue を読んで手元のドキュメントにまとめる、といった流れが作れます。

```text
myusername/my-app の未解決 Issue を調べて、対応方針の一覧を ~/Documents/issue-plan.md に保存して
```

## まとめ

- 旧 `@modelcontextprotocol/server-github` は開発が終了しました。現在は GitHub 公式の `github/github-mcp-server` を使います
- Claude Code と Codex はリモート版、Claude Desktop は Docker のローカル版が導入しやすい構成です
- fine-grained トークン、読み取り専用、ロックダウンで権限を絞って使います

新しい MCP 記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。

## 参考リンク

- [github/github-mcp-server - GitHub](https://github.com/github/github-mcp-server)
- [Claude 向けインストールガイド - github-mcp-server](https://github.com/github/github-mcp-server/blob/main/docs/installation-guides/install-claude.md)
- [Codex 向けインストールガイド - github-mcp-server](https://github.com/github/github-mcp-server/blob/main/docs/installation-guides/install-codex.md)
- [リモートサーバーの設定 - github-mcp-server](https://github.com/github/github-mcp-server/blob/main/docs/remote-server.md)
- [個人用アクセストークンを管理する - GitHub Docs](https://docs.github.com/ja/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- [GitHub MCP Exploited - Invariant Labs](https://invariantlabs.ai/blog/mcp-github-vulnerability)

次回は、デザインとコードを連携させる「[Figma MCP](./mcp-server-tutorial-06-figma)」を解説します。
