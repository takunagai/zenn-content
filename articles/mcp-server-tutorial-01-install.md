---
title: "【MCPのトリセツ #1】MCPの概要と導入方法"
emoji: "🐸"
type: "tech"
topics: ["mcp", "claude", "claudecode", "codex", "ai"]
published: true
---

このシリーズでは、Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を、サーバーごとに解説します。初回は MCP の概要と、Claude Desktop・Claude Code・Codex それぞれへの導入方法をまとめます。

:::message
**更新日: 2026-09-22**（初版: 2025-03-08）

旧版で紹介していた mcp-installer は更新が止まっているため、現在の公式の導入方法に差し替えています。
:::

### シリーズ目次

1. **MCPの概要と導入方法（この記事）**
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
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

## MCP とは

**MCP（Model Context Protocol）** は、AI アプリケーションと外部のツールやデータをつなぐための共通規格です。AI 単体では手元のファイルも、社内の Slack も、GitHub のリポジトリも直接は扱えません。MCP サーバーを間に置くと、AI がそれらを「ツール」として呼び出せるようになります。

- 指定したフォルダのファイルを読み書きする
- GitHub・Slack・Figma などのサービスを操作する
- Web ページの取得や検索を行う

MCP は 2024 年 11 月に Anthropic が公開し、2025 年 12 月に Linux Foundation 傘下の Agentic AI Foundation へ寄贈されました。いまは Claude だけでなく、ChatGPT / Codex、Cursor、VS Code など多くのクライアントが対応する、ベンダー中立の標準です。仕様は日付でバージョン管理されていて、現行は `2026-07-28` です。

## サーバーは 2 種類ある

導入方法を選ぶ前に、MCP サーバーに 2 つの形があることを押さえておくと迷いません。

| 種類 | 動く場所 | 接続方式 | 例 |
|---|---|---|---|
| ローカルサーバー | 自分の PC | stdio（標準入出力） | Filesystem、Blender、mcp-pandoc |
| リモートサーバー | サービス提供元のクラウド | Streamable HTTP + OAuth | GitHub、Figma、Slack |

1 年半前はローカルサーバーを `npx` や `uvx` で起動する方式がほとんどでした。現在は、サービス提供元が公式のリモートサーバーを用意し、URL を登録して OAuth でログインするだけで使える形が主流になっています。API キーを設定ファイルに書かずに済むので、まずリモート版の有無を確認するのが近道です。

なお、初期のリモート接続で使われていた HTTP+SSE 方式は非推奨になりました。古い記事で `sse` と書かれた設定を見かけたら、提供元の最新ドキュメントで Streamable HTTP の URL を確認してください。

## 導入方法の早見表

| クライアント | 方法 | 向いている場面 |
|---|---|---|
| Claude Desktop | コネクタ | リモートサーバーを使う。GUI だけで完結する |
| Claude Desktop | 拡張機能（.mcpb） | ローカルサーバーをワンクリックで入れる |
| Claude Desktop | 設定ファイルの編集 | 拡張機能として配布されていないローカルサーバーを使う |
| Claude Code | `claude mcp add` | ターミナルから追加する。プロジェクト単位で共有できる |
| Codex | `codex mcp add` | CLI・IDE 拡張・デスクトップアプリで設定を共有する |

以降の記事では、サーバーごとにこの中から使える方法を示します。

## Claude Desktop に導入する

### コネクタ（リモートサーバー）

サイドバーの「Customize」から「Connectors」を開き、「+」を押します。ディレクトリに載っているサービスは一覧から選んで OAuth 認証するだけです。ディレクトリにないリモートサーバーは「Add custom connector」で URL を登録します。

カスタムコネクタは Free プランでも 1 つまで使えます。Team / Enterprise では、先に組織のオーナーが「Organization settings > Connectors」で登録する必要があります。

カスタムコネクタは Anthropic の審査を経ていないサーバーにも接続できてしまいます。提供元が信頼できる URL だけを登録してください。

### 拡張機能（ローカルサーバー）

「Settings > Extensions」を開き、「Browse extensions」から使いたいものを選んで「Install」を押します。Claude Desktop には Node.js が同梱されているので、Node.js を別途入れる必要はありません。API キーなどの入力欄は OS のキーチェーン（macOS はキーチェーン、Windows は資格情報マネージャー）に暗号化して保存されます。

配布されている `.mcpb` ファイルを手元から入れる場合は、「Extensions」の「Advanced settings」にある「Install Extension…」を使います。

### 設定ファイルの編集（ローカルサーバー）

拡張機能になっていないサーバーは、従来どおり設定ファイルに書きます。メニューバーの「Claude > Settings...」から「Developer」タブを開き、「Edit Config」を押すとファイルが開きます。

- macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
- Windows: `%APPDATA%\Claude\claude_desktop_config.json`

書式は次のとおりです。複数のサーバーを使うときは `mcpServers` の中に並べます。

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/username/Desktop"
      ]
    }
  }
}
```

保存したら Claude Desktop を完全に終了して起動し直します。入力欄の左下にある「+」から「Connectors」にカーソルを合わせ、「Manage connectors」を開くと、接続中のサーバーとツールの一覧を確認できます。

うまく接続できないときは、ログを見ると原因が分かります。

```bash
tail -n 20 -f ~/Library/Logs/Claude/mcp*.log
```

## Claude Code に導入する

ターミナルで `claude mcp add` を実行します。ローカルサーバーは `--` の後ろに起動コマンドを書きます。

```bash
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/Desktop
```

環境変数が要るサーバーは `-e` で渡します。

```bash
claude mcp add my-server -e API_KEY=xxx -- npx my-mcp-server
```

リモートサーバーは `--transport http` と URL を指定します。OAuth が必要なサーバーは、Claude Code 内で `/mcp` を実行して認証します。

```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```

登録先は `-s`（`--scope`）で選べます。

| スコープ | 保存先 | 用途 |
|---|---|---|
| `local`（既定） | 自分の設定・現在のプロジェクトのみ | 試用、個人用のキーを含む設定 |
| `project` | リポジトリ直下の `.mcp.json` | チームで共有する |
| `user` | 自分の設定・全プロジェクト | どこでも使うサーバー |

Claude Desktop にすでに設定がある場合は、`claude mcp add-from-claude-desktop` で取り込めます（macOS と WSL のみ）。登録状況は `claude mcp list`、削除は `claude mcp remove <name>` です。

## Codex に導入する

Codex は `codex mcp add` で追加します。書式は Claude Code とよく似ています。

```bash
codex mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/Desktop
```

```bash
codex mcp add my-server --env API_KEY=xxx -- npx my-mcp-server
```

リモートサーバーは `--url` を使い、OAuth 対応のサーバーは `codex mcp login <name>` で認証します。

```bash
codex mcp add figma --url https://mcp.figma.com/mcp
```

設定は `~/.codex/config.toml` に保存され、Codex CLI・IDE 拡張・デスクトップアプリで共有されます。プロジェクト単位にしたい場合は、リポジトリ内の `.codex/config.toml` に書きます。直接編集するときの書式は次のとおりです。

```toml
[mcp_servers.filesystem]
command = "npx"
args = ["-y", "@modelcontextprotocol/server-filesystem", "/Users/username/Desktop"]

[mcp_servers.figma]
url = "https://mcp.figma.com/mcp"
```

トークンで認証するリモートサーバーは、`bearer_token_env_var = "環境変数名"` と書くと、トークン本体を設定ファイルに残さずに済みます。

## 前提ツール

ローカルサーバーの多くは、Node.js の `npx` か Python の `uvx` で起動します。

- **Node.js**: [公式サイト](https://nodejs.org/) から LTS 版を入れます（Claude Desktop の拡張機能だけを使うなら不要です）
- **uv**: Python 製のサーバーで使います。macOS は `brew install uv` で入ります。その他の環境は [uv の公式ドキュメント](https://docs.astral.sh/uv/getting-started/installation/) を参照してください

```bash
node --version
uv --version
```

バージョン番号が表示されれば準備完了です。

## API キーとトークンの扱い

MCP サーバーの設定では API キーやアクセストークンを扱う場面があります。私は次の 3 点を守っています。

- AI とのチャット欄にキーを貼りません。会話履歴に残るためです
- 設定ファイルに書いたキーは平文で保存されます。設定ファイルを Git にコミットしたり、画面共有で映したりしないよう注意が要ります
- OAuth で接続できるリモート版、OS のキーチェーンに保存する拡張機能、環境変数を参照する書き方を優先すると、平文のキーを減らせます

もう 1 点、MCP サーバーは AI に「操作する権限」を渡す仕組みです。外部から取り込んだ文章（Web ページ、Issue、メッセージなど）に紛れた指示を AI が実行してしまうプロンプトインジェクションの報告もあります。書き込みや送信を伴うツールは承認を都度確認する設定のままにし、権限は必要な範囲に絞るのが安全です。

## MCP とスキルの選び方

この 1 年半で、AI にできることを増やす手段として「スキル（Agent Skills）」も広まりました。スキルは、手順を書いた `SKILL.md` と補助スクリプトをまとめたフォルダです。Claude Code は `~/.claude/skills/`、Codex は `~/.agents/skills/` などに置くと、AI が必要な場面で読み込んで使います。

MCP サーバーとスキルは競合するものではなく、得意な場面が違います。

| 観点 | MCP サーバー | スキル |
|---|---|---|
| 実体 | ツールを公開するプログラム、またはリモートのサービス | 手順書（`SKILL.md`）とスクリプトのフォルダ |
| 得意な場面 | OAuth でログインする外部サービス、起動中のアプリとの接続、チャット型アプリからの利用 | CLI で完結する作業、決まった手順のあるワークフロー |
| コンテキストの消費 | 接続したサーバーのツール定義が載る（クライアントによっては遅延読み込みで軽減される） | 常時読まれるのは名前と説明だけ。本文は使うときに読まれる |
| 動く環境 | MCP 対応のクライアント全般 | コマンドを実行できるエージェント（Claude Code、Codex など） |

選ぶ基準は次のとおりです。

- GitHub・Figma・Slack のように、サービス側が公式のリモート MCP サーバーを用意していて OAuth で接続できるものは MCP を使います
- `yt-dlp`・`pandoc`・`gh` のように手元の CLI で済む作業は、Claude Code や Codex ならスキル（または CLI の直接実行）にします。MCP サーバーを常駐させる必要がありません
- Claude Desktop のようなチャット型アプリから手元のツールを使いたいときは、MCP サーバー（拡張機能）にします

このシリーズの各記事でも、スキルや CLI の方が向く場面にはその旨を書いています。

## 旧版で紹介していた mcp-installer について

旧版では、チャットで指示するだけで MCP サーバーを導入できる [mcp-installer](https://github.com/anaisbetts/mcp-installer) を紹介していました。このツールは 2024 年 11 月を最後に更新が止まっています。いまは拡張機能とコネクタが GUI での導入を担い、CLI も 1 行で追加できるようになったので、このシリーズでは mcp-installer を使わない手順に統一しました。

## MCP サーバーの探し方

| サイト | 特徴 |
|---|---|
| [公式 MCP Registry](https://registry.modelcontextprotocol.io/) | MCP プロジェクト公式のレジストリ。サーバーの公開元を名前空間で確認できる |
| [Claude のコネクタディレクトリ](https://claude.ai/directory) | Anthropic が審査したコネクタと拡張機能の一覧 |
| [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) | 公式リファレンス実装。旧サーバーは [servers-archived](https://github.com/modelcontextprotocol/servers-archived) に移動 |
| [Glama](https://glama.ai/mcp/servers) | オープンソースの MCP サーバーを横断検索できる |
| [PulseMCP](https://pulsemcp.com/) | MCP サーバーとクライアントのディレクトリ |
| [Smithery](https://smithery.ai/) | レジストリに加えてホスティングも提供する |

サービス名で探すときは、まずそのサービスの公式ドキュメントに MCP の案内がないか確認するのが確実です。同名のコミュニティ製サーバーが複数あることも多いので、公開元を確かめてから導入してください。

## まとめ

- MCP は AI と外部ツールをつなぐ共通規格で、現在はベンダー中立の標準になっています
- サーバーにはローカル（stdio）とリモート（Streamable HTTP + OAuth）があり、公式のリモート版があればそれが第一候補です
- Claude Desktop はコネクタと拡張機能、Claude Code は `claude mcp add`、Codex は `codex mcp add` で導入します
- 手元の CLI で済む作業は、Claude Code や Codex ならスキルの方が軽くなります。OAuth が要る外部サービスやチャット型アプリからの利用は MCP が向きます

新しい MCP 記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。

## 参考リンク

- [Model Context Protocol 公式サイト](https://modelcontextprotocol.io/)
- [Connect to local MCP servers - Model Context Protocol](https://modelcontextprotocol.io/docs/develop/connect-local-servers)
- [Getting Started with Local MCP Servers on Claude Desktop - Claude Help Center](https://support.claude.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop)
- [Getting started with custom connectors using remote MCP - Claude Help Center](https://support.claude.com/en/articles/11175166-getting-started-with-custom-connectors-using-remote-mcp)
- [Connect Claude Code to tools via MCP - Claude Code Docs](https://code.claude.com/docs/en/mcp)
- [Model Context Protocol - Codex ドキュメント](https://developers.openai.com/codex/mcp)
- [Extend Claude with skills - Claude Code Docs](https://code.claude.com/docs/en/skills)
- [Agent Skills - Codex ドキュメント](https://developers.openai.com/codex/skills)
- [MCP joins the Agentic AI Foundation - MCP Blog](https://blog.modelcontextprotocol.io/posts/2025-12-09-mcp-joins-agentic-ai-foundation/)

次回は、ローカルファイルを AI から扱えるようにする「[Filesystem MCP Server](./mcp-server-tutorial-02-filesystem)」を解説します。
