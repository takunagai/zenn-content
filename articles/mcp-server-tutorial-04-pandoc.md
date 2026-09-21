---
title: "【MCPのトリセツ #4】mcp-pandoc： AIでドキュメント形式を変換"
emoji: "🐸"
type: "tech"
topics: ["mcp", "pandoc", "claude", "claudecode", "codex"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズです。今回は、文書変換ツール Pandoc を AI から操作できるようにする mcp-pandoc を取り上げます。

:::message
**更新日: 2026-09-22**（初版: 2025-03-08）

旧版では「PDF 変換は開発中」と書いていましたが、現在は実装済みです。対応形式、テンプレート指定などの新しい引数、Claude Code・Codex での導入、CLI やスキルで代替する方法を追記しました。壊れていた記事内リンクも修正しています。
:::

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. **mcp-pandoc： AIでドキュメント形式を変換（この記事）**
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

## mcp-pandoc が役立つ場面

AI とのチャットでまとまった文章ができたとき、「これを Word 文書で渡したい」「PDF にして送りたい」と思うことがあります。[mcp-pandoc](https://github.com/vivekVells/mcp-pandoc) は、文書変換ツール [Pandoc](https://pandoc.org/) を AI から呼び出せるようにする MCP サーバーです。会話の中の文章も、手元のファイルも変換できます。

## 対応形式

| 形式 | 読み込み | 書き出し |
|---|---|---|
| Markdown、HTML、テキスト | ○ | ○ |
| Word（docx）、OpenDocument（odt） | ○ | ○ |
| EPUB | ○ | ○ |
| reStructuredText、LaTeX、Jupyter Notebook（ipynb） | ○ | ○ |
| PDF | × | ○ |
| PowerPoint（pptx） | × | ○ |

PDF と PowerPoint は書き出し専用です。PDF の内容を読み取りたい場合は、[Markdownify MCP](./mcp-server-tutorial-09-markdownfy) など別の手段を使います。

## セットアップ手順

### 前提ツールのインストール

Pandoc 本体と uv が必要です。PDF に変換する場合は TeX Live も入れます。TeX Live は数 GB あるので、PDF が不要なら省略できます。

```bash
# macOS
brew install pandoc
brew install uv

# PDF 変換を使う場合のみ
brew install texlive
```

Windows は [Pandoc のインストールページ](https://pandoc.org/installing.html) からインストーラーを入手し、PDF 変換には [MiKTeX](https://miktex.org/) か TeX Live を入れます。

### Claude Desktop

設定ファイル（開き方は[シリーズ #1](./mcp-server-tutorial-01-install) を参照）に次を追加し、Claude Desktop を再起動します。

```json
{
  "mcpServers": {
    "mcp-pandoc": {
      "command": "uvx",
      "args": ["mcp-pandoc"]
    }
  }
}
```

### Claude Code・Codex

```bash
claude mcp add mcp-pandoc -- uvx mcp-pandoc
```

```bash
codex mcp add mcp-pandoc -- uvx mcp-pandoc
```

## CLI やスキルで代替する方法

Claude Code や Codex はコマンドを実行できるので、Pandoc を直接呼べば MCP サーバーは要りません。

```bash
pandoc report.md -o report.docx
```

「Markdown を社内テンプレートの Word にする」のように決まった変換を繰り返すなら、テンプレートの場所と変換コマンドを書いたスキルを 1 つ用意しておくと、毎回の指示が短くなります。mcp-pandoc が向くのは、コマンドを実行できない Claude Desktop のようなチャット型アプリから変換したい場合です。選び方の全体像は[シリーズ #1](./mcp-server-tutorial-01-install) にまとめました。

## 使用例（プロンプト）

保存先は、ファイル名と拡張子まで含めたフルパスで指定します。フォルダだけの指定や拡張子の省略では変換に失敗します。

### 会話の内容を文書にする

```text
ここまでの内容を docx に変換して、/Users/yourname/Desktop/SampleDoc.docx として保存して
```

```text
この会議メモを Word 文書に変換して。見出しを整理し、目次を付けて、/Users/yourname/Documents/Business/meeting-report.docx として保存して
```

```text
今の内容を PDF に変換して、/Users/yourname/Desktop/summary.pdf として保存して
```

### 既存のファイルを変換する

```text
/Users/yourname/Documents/report.md を Word 形式に変換し、同じフォルダに report.docx として保存して
```

```text
/Users/yourname/Documents/manual.docx を Markdown に変換して、/Users/yourname/Documents/manual.md として保存して
```

### 電子書籍とスライド

```text
/Users/yourname/Documents/Books/my-novel.md を EPUB に変換して、/Users/yourname/Documents/Books/my-novel.epub として保存して
```

```text
この勉強会の構成案を PowerPoint に変換して、/Users/yourname/Desktop/study-session.pptx として保存して。見出しごとに 1 スライドにして
```

### 他の MCP サーバーと組み合わせる

[YouTube MCP](./mcp-server-tutorial-03-youtube) と組み合わせると、動画の要約をそのまま配布用の文書にできます。

```text
1. この YouTube 講義の内容を Markdown で要約して：https://www.youtube.com/watch?v=xxxxx
2. それを HTML に変換して、目次を付け、/Users/yourname/Documents/lectures/summary.html として保存して
```

## 見た目を整える引数

現行版では、Pandoc の次の機能を引数として指定できます。

| 引数 | 用途 |
|---|---|
| `reference_doc` | 書式の元になるテンプレート文書を指定する（docx、odt、pptx）。社内の Word テンプレートのフォントや見出しスタイルを引き継げる |
| `defaults_file` | 変換設定をまとめた YAML ファイルを指定する。目次、章番号、メタデータなどを毎回指示せずに済む |
| `filters` | Pandoc フィルターを適用する |

```text
/Users/yourname/Documents/report.md を、/Users/yourname/Templates/company.docx をテンプレートにして Word に変換し、/Users/yourname/Documents/report.docx として保存して
```

日本語の文書を PDF にするときは、LaTeX のエンジンと文書クラスを日本語対応のものにする必要があります。次のような YAML を用意して `defaults_file` に指定します。

```yaml
# ja-pdf.yaml
pdf-engine: lualatex
variables:
  documentclass: ltjsarticle
toc: true
```

## 使用上の注意点

- **フルパスで指定する**: 入力も出力も、ファイル名と拡張子を含む完全なパスが必要です
- **PDF は書き出し専用**: PDF を読み込んで他の形式にすることはできません
- **上書きに注意**: 出力先に同名のファイルがあると上書きされます
- **複雑なレイアウトは崩れる**: 段組みや図の回り込みなど、形式間で対応しない要素は変換で失われます。変換後のファイルは必ず開いて確認します

## まとめ

- mcp-pandoc は Pandoc を AI から呼び出す MCP サーバーです。PDF と PowerPoint の書き出しにも対応しました
- 導入は `uvx mcp-pandoc` です。Pandoc 本体と、PDF 変換には TeX Live が必要です
- Claude Code や Codex では `pandoc` コマンドを直接使う方が軽くなります。チャット型アプリから変換したいときに MCP サーバーが役立ちます

新しい MCP 記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。

## 参考リンク

- [vivekVells/mcp-pandoc - GitHub](https://github.com/vivekVells/mcp-pandoc)
- [Pandoc 公式サイト](https://pandoc.org/)
- [TeX Live 公式サイト](https://tug.org/texlive/)

次回は、GitHub のリポジトリを AI との会話で操作できる「[GitHub MCPサーバー](./mcp-server-tutorial-05-github)」を解説します。
