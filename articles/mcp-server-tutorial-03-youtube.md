---
title: "【MCPのトリセツ #3】YouTube MCPサーバー：動画の内容を取得"
emoji: "🐸"
type: "tech"
topics: ["mcp", "youtube", "claude", "claudecode", "codex"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズです。今回は YouTube MCP サーバーを取り上げます。動画の字幕を AI に読ませて、要約や分析を頼めるようになります。

:::message
**更新日: 2026-09-22**（初版: 2025-03-08）

導入方法を mcp-installer を使わない手順に差し替え、Claude Code・Codex での導入と、スキルで代替する方法を追記しました。旧版の末尾に紛れ込んでいた無関係なコマンドも削除しています。
:::

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. **YouTube MCPサーバー：動画の内容を取得（この記事）**
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

## YouTube MCP サーバーでできること

長い動画を最後まで見る時間が取れないことは多いですし、見ても細部は忘れます。[mcp-youtube](https://github.com/anaisbetts/mcp-youtube) は、`yt-dlp` を使って動画の字幕をテキストとして取得し、AI に渡す MCP サーバーです。

- 動画の内容を要約する
- 長い講義や解説動画から要点を抜き出す
- 英語の動画を日本語で要約する
- 複数の動画を比較する

取得するのは字幕のテキストだけです。映像そのものは AI に渡りません。

## セットアップ手順

### yt-dlp のインストール

字幕のダウンロードには [yt-dlp](https://github.com/yt-dlp/yt-dlp) を使います。

```bash
# macOS
brew install yt-dlp

# Windows
winget install yt-dlp
```

```bash
yt-dlp --version
```

バージョン番号が表示されれば準備完了です。YouTube 側の仕様変更で字幕を取得できなくなることがあるので、動かなくなったらまず `brew upgrade yt-dlp` で更新します。

### Claude Desktop

設定ファイル（開き方は[シリーズ #1](./mcp-server-tutorial-01-install) を参照）に次を追加し、Claude Desktop を再起動します。パッケージ名は `mcp-youtube` ではなく `@anaisbetts/mcp-youtube` です。

```json
{
  "mcpServers": {
    "youtube": {
      "command": "npx",
      "args": ["-y", "@anaisbetts/mcp-youtube"]
    }
  }
}
```

### Claude Code・Codex

```bash
claude mcp add youtube -- npx -y @anaisbetts/mcp-youtube
```

```bash
codex mcp add youtube -- npx -y @anaisbetts/mcp-youtube
```

## スキルで代替する方法

Claude Code や Codex はコマンドを実行できるので、`yt-dlp` を直接呼べば MCP サーバーなしで同じことができます。次のような `SKILL.md` を `~/.claude/skills/youtube-transcript/`（Codex は `~/.agents/skills/youtube-transcript/`）に置きます。

````markdown
---
name: youtube-transcript
description: YouTube の URL から字幕を取得して要約・分析する。YouTube の URL とともに要約や内容の確認を頼まれたときに使う。
---

1. 次のコマンドで字幕を一時フォルダに保存する

   ```bash
   yt-dlp --skip-download --write-subs --write-auto-subs \
     --sub-langs "ja,en" --sub-format vtt \
     -o "/tmp/yt/%(id)s" "<URL>"
   ```

2. 保存された .vtt ファイルを読み、タイムスタンプと重複行を除いて本文を取り出す
3. 依頼された形式（要約、要点、比較など）でまとめる
````

スキルは名前と説明だけが常時読み込まれ、本文は使うときに読まれます。常駐するプロセスもありません。Claude Desktop のようにコマンドを実行できないアプリでは、MCP サーバーの方を使います。選び方の全体像は[シリーズ #1](./mcp-server-tutorial-01-install) にまとめました。

## 使用例（プロンプト）

「YouTube の」と明示しなくても、URL を渡せば字幕を取得してくれます。

### 基本

```text
要約して
https://www.youtube.com/watch?v=oXq7trXF4aI
```

```text
要約して。英語の動画なら、日本語に翻訳してから要約して
https://www.youtube.com/watch?v=oXq7trXF4aI
```

```text
この動画について、次の形式で整理して
1. 3 行の概要
2. 主要なポイント（箇条書き）
3. 詳しい説明が要る部分
https://www.youtube.com/watch?v=xxxxx
```

```text
この動画から「料金プラン」に関する話だけを抜き出して
https://www.youtube.com/watch?v=xxxxx
```

### 学習

```text
この講義動画の内容を学習ノートの形式でまとめて。重要な概念は太字にし、例や応用も含めて
https://www.youtube.com/watch?v=xxxxx
```

```text
この動画の内容をもとに、練習問題を 5 問と解答を作って
https://www.youtube.com/watch?v=xxxxx
```

### 仕事

```text
この製品発表の動画から、新機能、価格、発売日、競合製品との比較点を表にまとめて
https://www.youtube.com/watch?v=xxxxx
```

```text
この技術チュートリアル動画から、手順書を作って。各ステップにはコマンドやコードも含めて
https://www.youtube.com/watch?v=xxxxx
```

```text
この動画の字幕をもとに、話し言葉を書き言葉に直して、読みやすい記事の形式にして
https://www.youtube.com/watch?v=xxxxx
```

### 複数動画の比較

```text
これら 3 つの動画から、共通するテーマと各動画独自の視点を比較したレポートを作って
https://www.youtube.com/watch?v=xxxxx
https://www.youtube.com/watch?v=yyyyy
https://www.youtube.com/watch?v=zzzzz
```

```text
対立する意見を述べているこの 2 つの動画を分析し、両者の主張、根拠、論理の強い点と弱い点を整理して
https://www.youtube.com/watch?v=xxxxx
https://www.youtube.com/watch?v=yyyyy
```

## Filesystem MCP との連携

[Filesystem MCP](./mcp-server-tutorial-02-filesystem) と組み合わせると、要約をそのまま手元のファイルに保存できます。

```text
この動画を要約し、ドキュメントフォルダの video-summary.md に保存して
https://www.youtube.com/watch?v=xxxxx
```

## 注意点と制限事項

- **字幕の有無**: 字幕（自動生成を含む）がない動画は処理できません
- **視覚情報**: 画面に映っているスライドやコードは読み取れません。字幕のテキストだけが対象です
- **長い動画**: 数時間の動画は字幕だけでも大きく、AI のコンテキスト上限に近づきます。前半と後半に分けて頼むと安定します
- **自動生成字幕の精度**: 固有名詞や専門用語は誤変換が混じります。数値や名称は動画本体で確認します
- **利用規約**: 取得した字幕の再配布は、動画の権利者と YouTube の利用規約に従います

## まとめ

- YouTube MCP サーバーは `yt-dlp` で字幕を取得して AI に渡します。Claude Desktop は設定ファイル、Claude Code と Codex は 1 行のコマンドで導入します
- コマンドを実行できる Claude Code や Codex では、`yt-dlp` を呼ぶスキルで代替でき、その方が軽くなります
- 対象は字幕のテキストだけです。映像の内容や、字幕のない動画は扱えません

新しい MCP 記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。

## 参考リンク

- [anaisbetts/mcp-youtube - GitHub](https://github.com/anaisbetts/mcp-youtube)
- [yt-dlp - GitHub](https://github.com/yt-dlp/yt-dlp)
- [Homebrew の yt-dlp](https://formulae.brew.sh/formula/yt-dlp)

次回は、Markdown や Word などの文書形式を変換できる「[mcp-pandoc](./mcp-server-tutorial-04-pandoc)」を解説します。
