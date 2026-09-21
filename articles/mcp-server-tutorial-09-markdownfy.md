---
title: "【MCPのトリセツ #9】Markdownify MCP Server: WebページやPDFをMarkdown文書化"
emoji: "🐸"
type: "tech"
topics: ["mcp", "markdown", "claude", "claudecode", "codex"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズです。今回は、Web ページ、PDF、Office 文書、音声などを Markdown に変換する Markdownify MCP Server を取り上げます。

:::message
**更新日: 2026-09-22**（初版: 2025-03-08）

インストール手順を現行の README（bun を使う方式）に合わせ、読み取り範囲を制限する設定を追記しました。旧版の「npm でもいけるはず」は誤りで、npm では配布されていません。あわせて、1 行で導入できる Microsoft 公式の markitdown-mcp と、CLI・スキルで代替する方法を紹介します。
:::

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. [Slack MCPサーバー：チームコミュニケーションを強化](./mcp-server-tutorial-07-slack)
8. [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
9. **Markdownify MCP Server: WebページやPDFをMarkdown文書化（この記事）**
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)
11. [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
12. [Blender MCP Server: 会話で Blender を操作し3Dモデルを作成](./mcp-server-tutorial-12-blender)
13. [Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行](./mcp-server-tutorial-13-perplexity)
14. [国土交通省がMCPサーバーを公開：AI時代のオープンデータ活用45選](./mcp-server-tutorial-14-milt-data)

資料: [ウェブ情報を取得するMCPの比較 (Fetch、Firecrawl、Markdownify、Perplexity)](./mcp-server-tutorial-reference-web-mcp)

---

## Markdownify MCP Server でできること

[zcaceres/markdownify-mcp](https://github.com/zcaceres/markdownify-mcp) は、さまざまな形式のコンテンツを Markdown に変換する MCP サーバーです。内部では Microsoft のオープンソースツール [MarkItDown](https://github.com/microsoft/markitdown) を使っています。

| ツール | 変換の対象 |
|---|---|
| `webpage-to-markdown` | 一般的な Web ページ |
| `pdf-to-markdown` | PDF ファイル |
| `docx-to-markdown`、`xlsx-to-markdown`、`pptx-to-markdown` | Office 文書 |
| `image-to-markdown` | 画像（メタデータ付き） |
| `audio-to-markdown` | 音声（文字起こし付き） |
| `youtube-to-markdown` | YouTube 動画の字幕 |
| `bing-search-to-markdown` | Bing の検索結果 |
| `get-markdown-file` | 既存の Markdown ファイルの取得 |

AI に渡す前に Markdown にしておくと、元のレイアウト情報が落ちて本文だけが残るので、長い資料でもコンテキストを節約できます。

> Fetch、Firecrawl、Markdownify、Perplexity の使い分けは、[ウェブ情報を取得するMCPの比較](./mcp-server-tutorial-reference-web-mcp)にまとめています。

## インストールと設定

### インストール

npm では配布されていないので、リポジトリをクローンしてビルドします。現行版はパッケージ管理に [bun](https://bun.sh/) を使います。Python 環境の用意に uv も必要です。

```bash
# 置き場所は任意。私は ~/tools/mcp-server にまとめている
cd ~/tools/mcp-server
git clone https://github.com/zcaceres/markdownify-mcp.git
cd markdownify-mcp

bun install
bun run build
```

`bun install` の途中で、プロジェクト内に Python の仮想環境（`.venv`）が作られ、`markitdown[all]` がインストールされます。

### Claude Desktop

設定ファイル（開き方は[シリーズ #1](./mcp-server-tutorial-01-install) を参照）に次を追加します。`args` にはビルドでできた `dist/index.js` の絶対パスを書きます。`~` でホームフォルダを表すとエラーになるので、`/Users/yourname/...` の形で書いてください。

```json
{
  "mcpServers": {
    "markdownify": {
      "command": "node",
      "args": ["/Users/yourname/tools/mcp-server/markdownify-mcp/dist/index.js"],
      "env": {
        "MD_ALLOWED_PATHS": "/Users/yourname/Documents:/Users/yourname/Downloads"
      }
    }
  }
}
```

`MD_ALLOWED_PATHS` は、ファイルを読み取れるフォルダを制限する設定です（macOS と Linux は `:`、Windows は `;` で区切ります）。未設定だと PC 内のどのファイルも読めてしまうので、指定しておくことをおすすめします。

### Claude Code・Codex

```bash
claude mcp add markdownify -- node /Users/yourname/tools/mcp-server/markdownify-mcp/dist/index.js
```

```bash
codex mcp add markdownify -- node /Users/yourname/tools/mcp-server/markdownify-mcp/dist/index.js
```

## 手早く入れるなら Microsoft 公式の markitdown-mcp

ビルドの手間を避けたい場合は、MarkItDown の開発元である Microsoft が公開している [markitdown-mcp](https://github.com/microsoft/markitdown/tree/main/packages/markitdown-mcp) が使えます。PyPI で配布されているので、uv が入っていれば 1 行です。

```bash
claude mcp add markitdown -- uvx markitdown-mcp
```

```json
{
  "mcpServers": {
    "markitdown": {
      "command": "uvx",
      "args": ["markitdown-mcp"]
    }
  }
}
```

| 観点 | Markdownify MCP | markitdown-mcp |
|---|---|---|
| 提供元 | コミュニティ（zcaceres） | Microsoft |
| 導入 | クローンしてビルド | `uvx markitdown-mcp` |
| ツール | 形式ごとに 10 個 | `convert_to_markdown(uri)` の 1 個 |
| 入力の指定 | ファイルパスや URL | `http:`・`https:`・`file:`・`data:` の URI |
| 読み取り範囲の制限 | `MD_ALLOWED_PATHS` | なし（README は Docker でのフォルダのマウントを推奨） |

変換エンジンは同じ MarkItDown なので、変換結果の質に大きな差は出ません。YouTube の字幕や Bing 検索を個別のツールとして使いたい、読み取り範囲をサーバー側で絞りたい、という場合は Markdownify が向きます。markitdown-mcp の README は、信頼できるローカルのエージェントと組み合わせて使うことを前提にしている点に注意してください。

## CLI やスキルで代替する方法

Claude Code や Codex なら、MarkItDown の CLI を直接呼べば MCP サーバーは要りません。

```bash
uvx --from 'markitdown[all]' markitdown report.pdf -o report.md
```

「資料を Markdown にして所定のフォルダに保存する」という決まった流れがあるなら、このコマンドと保存先のルールをスキルに書いておくと、常駐するサーバーなしで同じことができます。MCP サーバー版が向くのは、コマンドを実行できないチャット型アプリから使う場合です。選び方の全体像は[シリーズ #1](./mcp-server-tutorial-01-install) にまとめました。

## プロンプトのサンプル

### Web ページを変換する

```text
このページの内容を Markdown に変換して
https://example.com/blog/article-123
```

```text
次の複数の URL を Markdown に変換し、ページごとに見出しで区切って
- https://site1.com/article1
- https://site2.com/article2
- https://site3.com/article3
```

```text
この技術ブログから、コードスニペットと主要な見出しだけを抜き出して Markdown にして
https://tech-blog-example.com/tutorial/javascript-basics
```

### ファイルを変換する

```text
/Users/yourname/Documents/whitepaper.pdf を Markdown に変換して。目次の構造を保ち、表は Markdown のテーブルにして
```

```text
/Users/yourname/Documents/sales-2026Q2.xlsx を Markdown に変換して、シートごとに表として出力して
```

```text
/Users/yourname/Downloads/interview.m4a を文字起こしして Markdown にまとめて。話題が変わるところに見出しを付けて
```

### 変換と同時に整える

```text
この API ドキュメントを Markdown に変換して、次の形に整えて
- コードブロックは言語指定付きにする
- API エンドポイントは表にまとめる
- 重要な警告は太字にする
URL: https://api-docs-example.com/reference
```

```text
この英語の記事を Markdown に変換して、各セクションの冒頭に日本語で 2〜3 文の要約を付けて
https://longform-content.com/comprehensive-guide
```

```text
次の製品レビューのページから情報を取り出して、製品名、価格、主な機能、長所、短所を列にした比較表を Markdown で作って
- https://review-site.com/product1
- https://review-site.com/product2
- https://review-site.com/product3
```

変換そのものはサーバーが行い、絞り込みや整形は AI が変換結果に対して行います。長いページを丸ごと変換するとコンテキストを大きく消費するので、必要な部分が分かっているときは先に伝えておくと無駄がありません。

## 使用上の注意点

- **読み取り範囲を絞る**: `MD_ALLOWED_PATHS` を設定して、変換対象のフォルダだけを許可します
- **変換の精度**: 段組みの PDF、スキャン画像の PDF、複雑な表は崩れやすい形式です。重要な数値は元のファイルと突き合わせます
- **音声と画像**: 文字起こしと画像の解析には `markitdown[all]` の追加機能が必要です。公開されている Docker イメージには含まれていないので、この 2 つを使うならローカルにインストールします
- **取得した文章は外部の入力**: Web ページやファイルには AI への指示を装った文章が含まれている可能性があります。変換の直後に AI が想定外の操作をしようとしたら、承認せずに止めます

## まとめ

- Markdownify MCP は、Web ページ・PDF・Office 文書・音声などを Markdown にします。導入はクローンして `bun install`、`bun run build` です
- 手早く入れるなら Microsoft 公式の `uvx markitdown-mcp` が使えます。変換エンジンは同じ MarkItDown です
- Claude Code や Codex では `markitdown` コマンドの直接実行やスキルで代替できます

新しい MCP 記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。

## 参考リンク

- [zcaceres/markdownify-mcp - GitHub](https://github.com/zcaceres/markdownify-mcp)
- [microsoft/markitdown - GitHub](https://github.com/microsoft/markitdown)
- [markitdown-mcp - GitHub](https://github.com/microsoft/markitdown/tree/main/packages/markitdown-mcp)

次回は、ブックマークサービスを AI から使う「[Raindrop.io MCP Server](./mcp-server-tutorial-10-raindropio)」を解説します。
