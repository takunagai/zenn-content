---
title: "【MCPのトリセツ 資料】ウェブ情報を取得するMCPの比較 (Fetch、Firecrawl、Markdownify、Perplexity)"
emoji: "🐸"
type: "tech"
topics: ["mcp", "claude", "claudecode", "codex", "ai"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズの資料編です。Web の情報を取得する 4 つの MCP サーバー（Fetch、Firecrawl、Markdownify、Perplexity）の違いと使い分けをまとめます。

:::message
**更新日: 2026-09-22**（初版: 2025-03-11）

4 つのサーバーの現行の仕様に合わせて比較表を作り直しました。あわせて、AI 本体が Web 検索と Web 取得を標準で備えた現在の前提と、CLI・スキルで代替する選択肢を加えています。Firecrawl の Deep Research は廃止され、Perplexity は Sonar から Agent API へ移行しています。
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
9. [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)
11. [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
12. [Blender MCP Server: 会話で Blender を操作し3Dモデルを作成](./mcp-server-tutorial-12-blender)
13. [Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行](./mcp-server-tutorial-13-perplexity)
14. [国土交通省がMCPサーバーを公開：AI時代のオープンデータ活用45選](./mcp-server-tutorial-14-milt-data)

資料: **ウェブ情報を取得するMCPの比較 (Fetch、Firecrawl、Markdownify、Perplexity)（この記事）**

---

## まず標準機能で足りるかを確かめる

旧版を書いた 2025 年 3 月には、Claude Desktop に Web ページを読ませるだけでも MCP サーバーが必要でした。現在は Claude も Codex も Web 検索と Web 取得を標準で備えています。公開されている普通のページを読む、最近のニュースを調べる、といった用途は標準機能で足ります。

MCP サーバーを足す価値があるのは、標準機能では届かない次のような場面です。

| 場面 | 向いているサーバー |
|---|---|
| JavaScript で描画されるページ、クリックやログイン後の表示を取得したい | Firecrawl |
| サイト全体をクロールして、まとめて取り込みたい | Firecrawl |
| `localhost` や社内ネットワークのページを読ませたい | Fetch |
| PDF、Office 文書、音声を Markdown にしたい | Markdownify |
| 出典付きの調査レポートをまとめて取りたい | Perplexity |

## 機能比較表

| 観点 | Fetch | Firecrawl | Markdownify | Perplexity |
|---|---|---|---|---|
| 提供元 | MCP 公式リファレンス実装 | Firecrawl 社 | コミュニティ | Perplexity 社 |
| 導入 | `uvx mcp-server-fetch` | `npx -y firecrawl-mcp`、またはリモート版 | クローンしてビルド | `npx -y @perplexity-ai/mcp-server` |
| 費用 | 無料 | 月 1,000 クレジットまで無料、以降は有料 | 無料 | 従量課金 |
| API キー | 不要 | 必要（リモート版は OAuth） | 不要 | 必要 |
| 単一ページの取得 | ○ | ○ | ○ | ×（検索が入口） |
| JavaScript で描画されるページ | × | ○ | × | ─ |
| 複数ページのクロール | × | ○ | × | × |
| クリックや入力を伴う取得 | × | ○ | × | × |
| Web 検索 | × | ○ | Bing の検索結果を取得 | ○ |
| 出典付きの調査 | × | Agent 機能 | × | ○（research） |
| PDF・Office 文書の変換 | × | PDF などの解析 | ○ | × |
| 音声の文字起こし | × | × | ○ | × |
| `localhost`・社内ページ | ○ | × | ○ | × |

## 4 つのサーバーの特徴

### Fetch MCP

[Fetch MCP Server](./mcp-server-tutorial-11-fetch) は、URL を 1 つ受け取り、ページを Markdown にして返すだけの小さなサーバーです。手元の PC からアクセスするので、開発中のサイトや社内のページを AI に読ませる用途に向きます。JavaScript は実行しません。

### Firecrawl MCP

[Firecrawl MCP](./mcp-server-tutorial-08-firecrawl) は、スクレイピング、サイトマップ取得、クロール、検索、ブラウザ操作、自律的に調べる Agent 機能までを備えたサーバーです。標準の Web 取得で中身が空になるページや、サイトを丸ごと取り込みたいときの選択肢です。旧版で触れていた Deep Research は廃止され、検索と Agent 機能に置き換わりました。

### Markdownify MCP

[Markdownify MCP Server](./mcp-server-tutorial-09-markdownfy) は、Web ページに加えて、PDF、Office 文書、画像、音声、YouTube の字幕を Markdown に変換します。変換エンジンは Microsoft の MarkItDown で、同じエンジンを使う Microsoft 公式の `markitdown-mcp` も 1 行で導入できます。

### Perplexity MCP

[Perplexity MCP Server](./mcp-server-tutorial-13-perplexity) は、Perplexity の検索と、検索を踏まえた回答・推論・調査を呼び出します。ツールは `perplexity_search`・`perplexity_ask`・`perplexity_reason`・`perplexity_research` の 4 つです。Sonar のモデルを直接指定する旧方式は終了し、Agent API のプリセットに移行しました。

## シナリオ別の使い分け

| やりたいこと | 選ぶもの |
|---|---|
| 公開ページを 1 つ読んで要約する | 標準の Web 取得 |
| 最新のニュースや動向を調べる | 標準の Web 検索。出典付きでまとめたいなら Perplexity |
| 開発中のサイト（`localhost`）の表示内容を確認する | Fetch |
| 標準の取得では本文が空になるページを読む | Firecrawl |
| ドキュメントサイトをまとめて取り込む | Firecrawl（map で URL を確認してから crawl） |
| 料金表のタブを切り替えてから取得する | Firecrawl（interact） |
| PDF や Word、PowerPoint を AI に読ませる | Markdownify |
| 会議の録音を文字起こしして要約する | Markdownify |
| 複数の情報源を突き合わせた調査レポートを作る | Perplexity（research）、または Firecrawl の Agent |
| 論文を探して読む | Firecrawl（research 系ツール）、または Perplexity |

組み合わせて使う場面もあります。Perplexity で情報源を見つけ、その中の重要なページを Firecrawl で全文取得し、[Filesystem MCP](./mcp-server-tutorial-02-filesystem) で手元に保存する、という流れです。

## CLI やスキルで代替する選択肢

Claude Code や Codex のようにコマンドを実行できる環境では、MCP サーバーを常駐させずに同じことができる場合があります。

| サーバー | CLI・スキルでの代替 |
|---|---|
| Fetch | `curl` で取得して読む |
| Firecrawl | 公式の CLI とスキル（`npx -y firecrawl-cli@latest init --all --browser`） |
| Markdownify | MarkItDown の CLI（`uvx --from 'markitdown[all]' markitdown file.pdf -o file.md`） |
| Perplexity | MCP サーバーを使う（API を直接呼ぶスクリプトを書くより手間が少ない） |

MCP サーバーは、接続するとツールの定義がコンテキストに載ります。Web 取得系を 4 つとも入れると、ツールの数は 40 個を超えます。スキルは名前と説明だけが常時読み込まれ、本文は使うときに読まれるので、使う頻度の低いものほどスキルや CLI に寄せる方が軽くなります。チャット中心の Claude Desktop ではコマンドを実行できないので、MCP サーバーの方を使います。選び方の全体像は[シリーズ #1](./mcp-server-tutorial-01-install) にまとめました。

## AI に使い分けを伝える指示の例

複数の取得手段を入れると、AI がどれを使うか迷ったり、有料のツールを不用意に呼んだりします。優先順位を書いておくと動きが安定します。Claude Desktop は個人設定（カスタム指示）、Claude Code は `CLAUDE.md`、Codex は `AGENTS.md` に書きます。

```markdown
## Web の情報を取得するときの優先順位

1. まず標準の Web 検索・Web 取得を使う
2. 次の場合だけ「fetch」を使う
   - localhost や社内のページを読むとき
   - HTML をそのまま確認したいとき
3. 次の場合だけ「markdownify」を使う
   - PDF、Office 文書、画像、音声を Markdown にするとき
4. 次の場合だけ、実行前に確認を取ってから「firecrawl」を使う
   - 標準の取得で本文が取れないページ
   - 複数ページのクロール、操作を伴う取得
   - 「Firecrawl で」と指示があるときは確認不要
5. 次の場合だけ、実行前に確認を取ってから「perplexity」を使う
   - 出典付きの調査レポートが必要なとき
   - 「Perplexity で」と指示があるときは確認不要

firecrawl のクロールと Agent、perplexity の research は費用がかかる。対象と上限を先に示すこと。
```

## まとめ

- 公開ページの閲覧と一般的な検索は、AI 本体の標準機能で足りるようになりました
- Fetch は `localhost` と社内ページ、Firecrawl は JavaScript 描画・クロール・操作、Markdownify はファイルの変換、Perplexity は出典付きの調査が持ち場です
- Claude Code や Codex では、CLI やスキルで代替できるものが多くあります。有料のツールは、使う条件を指示に書いておきます

## 関連記事

- [Fetch MCP Server: ウェブコンテンツを取得・処理](./mcp-server-tutorial-11-fetch)
- [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
- [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
- [Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行](./mcp-server-tutorial-13-perplexity)
