---
title: "【MCPのトリセツ #2】Filesystem MCP Server： AIでローカルファイルを扱う"
emoji: "🐸"
type: "tech"
topics: ["mcp", "claude", "claudecode", "codex", "ai"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズです。今回は、AI が手元のファイルを読み書きできるようになる「Filesystem MCP Server」を取り上げます。

:::message
**更新日: 2026-09-22**（初版: 2025-03-08）

導入方法を現行の内容（拡張機能・設定ファイル・CLI）に更新し、ツール一覧を最新の README に合わせました。旧版で「削除もできる」と書いていましたが、このサーバーに削除のツールはありません。訂正します。
:::

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. **Filesystem MCP Server： AIでローカルファイルを扱う（この記事）**
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

## Filesystem MCP Server が役立つ場面

チャット型の AI は、そのままでは手元の PC にあるファイルを開けません。ファイルを 1 つずつアップロードする方法はありますが、フォルダをまたいだ作業や、結果をファイルに書き戻す作業には向きません。

Filesystem MCP Server は、許可したフォルダの中に限って、AI にファイル操作をさせる仕組みです。

- 「このフォルダのメモを全部読んで要約して」と直接指示できる
- 複数のファイルをまとめて読み、まとめて編集できる
- AI が作った文章やデータを、そのままファイルとして保存できる

ファイルの中身は AI に渡すために Anthropic などの LLM サーバーへ送信されます。「クラウドにアップロードしないから機密文書でも安全」という意味ではない点に注意してください。

なお、Claude Code と Codex はもともと作業フォルダ内のファイルを読み書きできるので、このサーバーは要りません。主に Claude Desktop のようなチャット型のアプリで使うものです。

## 提供されるツール

| 分類 | ツール |
|---|---|
| 読み取り | `read_text_file`、`read_media_file`、`read_multiple_files` |
| 書き込み・編集 | `write_file`、`edit_file`、`create_directory`、`move_file` |
| 一覧・検索 | `list_directory`、`list_directory_with_sizes`、`directory_tree`、`search_files` |
| 情報取得 | `get_file_info`、`list_allowed_directories` |

削除のツールはありません。ただし `write_file` は既存のファイルを上書きでき、`move_file` は移動と名前の変更ができるので、内容を失う操作は起こりえます。

## セットアップ手順

### Claude Desktop：拡張機能で入れる

「Settings > Extensions」を開き、「Browse extensions」から Anthropic 製の「Filesystem」を選んで「Install」を押します。インストール時に、アクセスを許可するフォルダを選びます。Node.js のインストールも設定ファイルの編集も不要です。

2026 年 9 月時点で、この拡張機能のツールがスキーマ検証エラーで呼び出せないという不具合が [GitHub の Issue](https://github.com/anthropics/claude-code/issues/94351) に報告されています。同じ症状が出た場合は、次の設定ファイル方式を試してください。

### Claude Desktop：設定ファイルで入れる

[Node.js](https://nodejs.org/) を入れたうえで、設定ファイル（開き方は[シリーズ #1](./mcp-server-tutorial-01-install) を参照）に次を追加します。`args` の 3 つ目以降が、アクセスを許可するフォルダです。複数指定できます。

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/yourname/Documents",
        "/Users/yourname/Projects"
      ]
    }
  }
}
```

Windows の場合は `C:\\Users\\yourname\\Desktop` のように、バックスラッシュを 2 つ重ねて書きます。旧版では先に `npm install -g` を実行していましたが、`npx -y` が自動で取得するので不要です。

保存したら Claude Desktop を完全に終了して起動し直します。

### Claude Code・Codex で使う場合

作業フォルダの外にあるフォルダを AI に触らせたいときなど、あえて使う場合のコマンドです。

```bash
claude mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/Documents
```

```bash
codex mcp add filesystem -- npx -y @modelcontextprotocol/server-filesystem ~/Documents
```

### 動作確認

```text
/Users/yourname/Desktop にあるファイルの一覧を取得して、ファイル名を列挙して
```

フォルダの中身が返ってくれば成功です。操作のたびに承認を求められるので、内容を見てから許可します。

## 便利に使うテクニック

### フォルダに呼び名を付ける

毎回フルパスを書くのは手間なので、Claude の個人設定（カスタム指示）の欄に対応表を書いておきます。

```text
ローカルのフォルダは以下とします。
- 指定がない場合: '~/AI-Workspace'
- プロジェクトフォルダ: '~/Projects'
- ダウンロードフォルダ: '~/Downloads'
- デスクトップフォルダ: '~/Desktop'
```

こうしておくと、「デスクトップフォルダのファイル一覧を取得して」で通じます。既定の作業用フォルダ（上の例では `AI-Workspace`）を決めておくと、「今の内容をローカルに保存して」だけで保存先が決まります。ここに書くフォルダは、サーバー側で許可したフォルダの範囲内にしてください。

## 気をつけること

- 許可するフォルダは必要最小限にします。ホームフォルダ全体やシステムのフォルダは指定しません
- 上書きと移動ができるので、重要なフォルダは Git などでバージョン管理しておくと、元に戻せます
- 大量のファイルを一度に読ませると、応答が遅くなり、AI が扱えるコンテキストの上限も超えます。対象のファイルを作業用フォルダに集めてから指示するのが確実です
- ファイル一覧を尋ねたとき、AI が返答の中でファイル名を省略したり言い換えたりすることがあります。正確な名前が要るときは、その旨を指示に含めます
- 外部から入手したファイルには、AI への指示を装った文章が含まれている可能性があります。ファイルを読んだ直後に AI が想定外の操作をしようとしたら、承認せずに止めます

## プロンプトのサンプル

### 文書の整理と作成

```text
ドキュメントフォルダ/AI/MCP にある "MCPサーバー 01" 〜 "MCPサーバー 07" の記事を読んで、内容と構成を把握して
```

```text
把握した 7 記事を、次の条件で統一感のある形にリライトして。バックアップは別の場所に取ってあるので、上書きしてよい。

- MCP を初めて使う人に向けた技術ブログ記事
- 冒頭にシリーズ記事である旨と目次を置く
- 1 文を短くし、手順は番号付きリストにする
```

このシリーズの初版は、実際にこの方法で下書きを整えました。

### データの分析

```text
~/Projects/sales-data/ の CSV ファイルを分析して売上の傾向を調べ、結果を ~/Documents/reports/sales-report.md に保存して
```

### フォルダの整理

```text
ダウンロードフォルダのファイルを種類別（画像、PDF、その他）に分類する案を出して。移動は、私が案を承認してから実行して
```

移動や上書きを伴う作業は、先に計画だけを出させて、確認してから実行させると事故が減ります。

## 他の MCP サーバーとの連携

Filesystem MCP Server は「保存先」として他のサーバーと組み合わせやすい存在です。

- [YouTube MCP](./mcp-server-tutorial-03-youtube): 動画の字幕を要約して、ファイルに保存する
- [GitHub MCP](./mcp-server-tutorial-05-github): Issue の一覧を読んで、対応方針を手元の文書にまとめる
- [mcp-pandoc](./mcp-server-tutorial-04-pandoc): 保存した Markdown を Word や PDF に変換する

## まとめ

- Filesystem MCP Server は、許可したフォルダの中で AI にファイルの読み書きをさせる公式リファレンス実装です
- Claude Desktop では拡張機能か設定ファイルで導入します。Claude Code と Codex は標準でファイルを扱えるので、基本的に不要です
- 削除のツールはないが、上書きと移動はできます。許可フォルダを絞り、バージョン管理と併用します

新しい MCP 記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。

## 参考リンク

- [Filesystem MCP Server - modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem)
- [Connect to local MCP servers - Model Context Protocol](https://modelcontextprotocol.io/docs/develop/connect-local-servers)
- [Getting Started with Local MCP Servers on Claude Desktop - Claude Help Center](https://support.claude.com/en/articles/10949351-getting-started-with-local-mcp-servers-on-claude-desktop)

次回は、YouTube 動画の字幕を AI に読ませる「[YouTube MCPサーバー](./mcp-server-tutorial-03-youtube)」を解説します。
