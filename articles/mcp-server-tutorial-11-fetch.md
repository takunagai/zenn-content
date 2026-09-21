---
title: "【MCPのトリセツ #11】Fetch MCP Server: ウェブコンテンツを取得・処理"
emoji: "🐸"
type: "tech"
topics: ["mcp", "fetch", "claude", "claudecode", "codex"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズです。今回は、Web ページの内容を取得して Markdown で AI に渡す Fetch MCP Server を取り上げます。

:::message
**更新日: 2026-09-22**（初版: 2025-03-11）

導入手順に Claude Code と Codex を追加し、AI 本体の Web 取得機能が標準になった現在の位置づけを書き足しました。参考リンクにあった公式サイトの URL（`.ai` ドメイン）は無関係なページに変わっていたため、正しい `modelcontextprotocol.io` に修正しています。
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
11. **Fetch MCP Server: ウェブコンテンツを取得・処理（この記事）**
12. [Blender MCP Server: 会話で Blender を操作し3Dモデルを作成](./mcp-server-tutorial-12-blender)
13. [Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行](./mcp-server-tutorial-13-perplexity)
14. [国土交通省がMCPサーバーを公開：AI時代のオープンデータ活用45選](./mcp-server-tutorial-14-milt-data)

資料: [ウェブ情報を取得するMCPの比較 (Fetch、Firecrawl、Markdownify、Perplexity)](./mcp-server-tutorial-reference-web-mcp)

---

## Fetch MCP Server でできること

[Fetch MCP Server](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch) は、MCP の公式リファレンス実装の 1 つです。提供するツールは `fetch` の 1 つだけです。

| 引数 | 内容 |
|---|---|
| `url` | 取得する URL |
| `max_length` | 返す文字数の上限（既定は 5000） |
| `start_index` | 何文字目から返すか。長いページを続きから読むときに使う |
| `raw` | `true` にすると、Markdown に変換せず HTML のまま返す |

長いページは `max_length` ごとに区切って返され、AI が `start_index` を進めながら続きを読みます。

## いま導入する意味があるか

旧版を書いた 2025 年 3 月の時点では、Claude Desktop に Web ページを読ませる手段としてこのサーバーが手軽でした。現在は Claude も Codex も Web 検索と Web 取得を標準で備えているので、公開されている普通のページを読むだけなら、このサーバーは要りません。

それでも役に立つのは、次のような場面です。

- **手元の PC から取得したい**: 標準の Web 取得は AI の提供元のサーバーからアクセスします。Fetch MCP Server は自分の PC からアクセスするので、社内ネットワークのページや、`localhost` で動かしている開発中のサイトを読ませられます
- **User-Agent を変えたい**: モバイル向けの表示を確認したいときなどに使えます
- **HTML をそのまま見たい**: `raw` を指定すると、変換前の HTML を取得できます

> Fetch、Firecrawl、Markdownify、Perplexity の使い分けは、[ウェブ情報を取得するMCPの比較](./mcp-server-tutorial-reference-web-mcp)にまとめています。

## インストールと設定

Python 製のサーバーで、uv が入っていれば起動できます（uv の導入は[シリーズ #1](./mcp-server-tutorial-01-install) を参照）。

### Claude Desktop

設定ファイルに次を追加し、Claude Desktop を再起動します。

```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  }
}
```

### Claude Code・Codex

```bash
claude mcp add fetch -- uvx mcp-server-fetch
```

```bash
codex mcp add fetch -- uvx mcp-server-fetch
```

Claude Code と Codex はコマンドを実行できるので、`curl` で取得して読む方法でも同じことができます。決まったサイトを決まった形で取得する作業を繰り返すなら、取得コマンドと整形の手順をスキルにまとめる選択肢もあります（選び方は[シリーズ #1](./mcp-server-tutorial-01-install) を参照）。

### オプション

`args` に次の指定を追加できます。

| オプション | 内容 |
|---|---|
| `--ignore-robots-txt` | robots.txt を無視する。既定では、AI が自発的に行う取得は robots.txt に従い、ユーザーが指示した取得は従わない |
| `--user-agent=文字列` | User-Agent を変更する |
| `--proxy-url=URL` | プロキシを経由する |

既定の User-Agent は、取得のきっかけによって 2 種類あります。

- AI が自発的に取得した場合: `ModelContextProtocol/1.0 (Autonomous; +https://github.com/modelcontextprotocol/servers)`
- ユーザーの指示で取得した場合: `ModelContextProtocol/1.0 (User-Specified; +https://github.com/modelcontextprotocol/servers)`

iPhone として取得する設定例です。

```json
{
  "mcpServers": {
    "fetch": {
      "command": "uvx",
      "args": [
        "mcp-server-fetch",
        "--user-agent=Mozilla/5.0 (iPhone; CPU iPhone OS 18_0 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/18.0 Mobile/15E148 Safari/604.1"
      ]
    }
  }
}
```

## プロンプトのサンプル

標準の Web 取得が先に使われることがあるので、このサーバーを使わせたいときは「fetch で」と添えます。

```text
fetch で http://localhost:4321/ の内容を取得して、見出しの構造を一覧にして
```

```text
fetch で https://example.com/docs/api を取得して、エンドポイントの一覧を表にまとめて
```

```text
fetch で https://example.com/ を HTML のまま取得して、title と meta description、OGP の設定を確認して
```

```text
この長い記事を最後まで読んで、章ごとに要約して。途中で切れたら続きを取得して
https://example.com/long-article
```

## 使用上の注意点

- **内部ネットワークにもアクセスできる**: このサーバーは手元の PC からアクセスするため、ローカルや社内の IP アドレスにも届きます。README でもセキュリティ上の注意として明記されています。外部のページに書かれた指示に従って AI が内部の URL を取得する、といった動きには注意が要ります
- **JavaScript は実行されない**: 取得するのはサーバーが返す HTML です。JavaScript で描画されるページは中身が空に見えます。その場合は [Firecrawl MCP](./mcp-server-tutorial-08-firecrawl) などを使います
- **robots.txt と利用規約**: `--ignore-robots-txt` は、自分が管理するサイトの確認など、問題のない場面に限って使います

## まとめ

- Fetch MCP Server は、Web ページを Markdown にして AI に渡す公式リファレンス実装です。導入は `uvx mcp-server-fetch` です
- AI 本体が Web 取得を備えた現在は、`localhost` や社内ページの取得、User-Agent の変更、HTML の直接確認が主な用途になります
- 手元の PC からアクセスする仕組みなので、内部ネットワークに届く点を理解して使います

新しい MCP 記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。

## 参考リンク

- [Fetch MCP Server - modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch)
- [Model Context Protocol 公式サイト](https://modelcontextprotocol.io/)

次回は、会話で Blender を操作する「[Blender MCP Server](./mcp-server-tutorial-12-blender)」を解説します。
