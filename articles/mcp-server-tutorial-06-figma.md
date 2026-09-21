---
title: "【MCPのトリセツ #6】Figma MCP：デザインとコードを効率的に連携"
emoji: "🐸"
type: "tech"
topics: ["mcp", "figma", "claude", "claudecode", "codex"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズです。今回は Figma 公式の MCP サーバーを取り上げます。AI が Figma のデザインデータを直接読み取り、正確な数値に基づいてコードを書けるようになります。

:::message
**更新日: 2026-09-22**（初版: 2025-03-08）

旧版はコミュニティ製の `figma-developer-mcp` だけを紹介していましたが、その後 Figma 公式の MCP サーバーが登場したため、公式版を中心に書き直しています。
:::

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. **Figma MCP：デザインとコードを効率的に連携（この記事）**
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

## Figma MCP で解決できること

Figma のデザインをコードに起こすとき、色やサイズをスクリーンショットから目で拾うと、どうしてもずれが出ます。Figma MCP サーバーを使うと、AI がレイヤー構造・変数・コンポーネントの情報を直接取得するので、推測ではなく実データからコードを生成できます。

- フレームやコンポーネントを指定して、実装コードを生成する
- カラー・タイポグラフィ・スペーシングなどの変数（デザイントークン）を取り出す
- Code Connect で、Figma のコンポーネントと既存のコードコンポーネントを対応付ける
- AI から Figma のキャンバスへ、フレームやコンポーネントを書き戻す

最後の書き込み機能は公式版で追加されたもので、旧版の記事にはなかった使い方です。

## 公式版は 2 種類ある

| 種類 | 接続先 | 特徴 |
|---|---|---|
| リモートサーバー | `https://mcp.figma.com/mcp` | Figma デスクトップアプリが不要。機能が最も多い。Figma の推奨 |
| デスクトップサーバー | `http://127.0.0.1:3845/mcp` | Figma デスクトップアプリ内で起動する。選択中のレイヤーを対象にできる |

Figma は公式ドキュメントでリモートサーバーを強く推奨しているので、この記事でもリモート版で進めます。デスクトップ版は、組織の方針でローカル接続が求められる場合の選択肢です。

リモート版はすべてのプラン・シートで接続できますが、ツールの呼び出し回数にシートごとの上限があります。View・Collab シートは月に数回から 20 回までと少なく、日常的に使うには Dev または Full シートが要ります。上限の数値はプランによって異なるので、[公式の Rate limits & access](https://developers.figma.com/docs/figma-mcp-server/rate-limits-access/) で確認してください。

## セットアップ手順

API キーの発行は不要です。どのクライアントも、Figma アカウントでの OAuth 認証で接続します。

### Claude Code

公式プラグインを入れる方法が推奨されています。MCP サーバーの設定に加えて、Figma 向けのスキルも一緒に入ります。

```bash
claude plugin install figma@claude-plugins-official
```

MCP サーバーだけを手動で登録する場合は次のとおりです。全プロジェクトで使うなら `--scope user` を付けます。

```bash
claude mcp add --transport http figma https://mcp.figma.com/mcp
```

登録後、Claude Code 内で `/mcp` を実行し、`figma` を選んで「Authenticate」に進みます。ブラウザで「Allow Access」を押すと接続されます。

### Codex

Codex アプリの「Plugins」メニューから Figma プラグインを入れる方法と、CLI で登録する方法があります。

```bash
codex mcp add figma --url https://mcp.figma.com/mcp
```

実行後に表示される案内に従って、ブラウザで認証します。

### Claude Desktop

Figma が提供する公式コネクタがあります。サイドバーの「Customize」から「Connectors」を開き、「+」を押して Figma を選び、OAuth 認証を済ませます。[コネクタのページ](https://claude.com/connectors/figma)から追加することもできます。

## 基本的な使い方（プロンプト）

リモート版は、Figma のフレームやレイヤーへのリンクを渡して使います。Figma 上で対象を右クリックし、「Copy/Paste as」から「Copy link to selection」でリンクを取得します。リンクには `node-id=123-456` の形で対象のノード ID が含まれています。

```text
この Figma フレームを React + Tailwind CSS で実装して
https://www.figma.com/design/xxxxXXXX/ProjectName?node-id=123-456
```

```text
このボタンコンポーネントの色、サイズ、フォント、角丸の値を教えて
https://www.figma.com/design/xxxxXXXX/ProjectName?node-id=789-012
```

ファイル全体ではなく、フレームやセクション単位でリンクを渡すのがコツです。対象が大きいと取得するデータ量が増え、AI のコンテキストを圧迫します。

## 実践的な活用例

### デザイントークンを CSS 変数にする

```text
このフレームで使われている変数（カラー、タイポグラフィ、スペーシング）を取得して、CSS カスタムプロパティとして出力して
```

[Filesystem MCP](./mcp-server-tutorial-02-filesystem) と組み合わせるか、Claude Code・Codex から実行すれば、そのままファイルに保存できます。

```text
取得した変数を src/styles/tokens.css に保存して
```

### バリアントを持つコンポーネントを実装する

```text
このボタンコンポーネントを分析して、class-variance-authority（CVA）でバリアント（サイズ: sm, md, lg、種類: primary, secondary, outline, ghost）を持つ Tailwind CSS のボタンを実装して
```

### 既存のコンポーネントを使って実装する

```text
このフレームを実装して。src/components/ui にある既存のコンポーネントを優先して使い、足りないものだけ新規に作って
```

Code Connect でコンポーネントの対応付けを済ませておくと、AI は Figma のコンポーネントに対応するコード側のコンポーネントを把握した状態で実装します。デザインシステムを運用しているチームでは、ここが品質の分かれ目になります。

### 仕様書と一貫性チェック

```text
このヘッダーコンポーネントの仕様（サイズ、色、間隔、フォント）を Markdown の表にまとめて
```

```text
このフレームで、デザインシステムの変数を使わず直接指定されている色やフォントサイズを洗い出して
```

### コードから Figma へ書き戻す

```text
この画面の実装（http://localhost:3000/settings）をもとに、Figma に新しいデザインファイルを作ってレイヤーとして再現して
```

```text
このユーザー登録フローを FigJam のフローチャートにして
```

書き込み系のツールは実際に Figma のファイルを変更します。チームの共有ファイルに対して使う前に、下書き用のファイルで挙動を確かめておくのが安全です。

## コミュニティ版 Framelink について

旧版で紹介していた `figma-developer-mcp` は、[Framelink](https://github.com/GLips/Figma-Context-MCP) というコミュニティ製のサーバーで、現在も開発が続いています。Figma の個人アクセストークンで動き、レイアウトとスタイルの情報を絞り込んで AI に渡す設計です。

```json
{
  "mcpServers": {
    "framelink-figma": {
      "command": "npx",
      "args": ["-y", "figma-developer-mcp", "--figma-api-key=YOUR_FIGMA_API_KEY", "--stdio"]
    }
  }
}
```

公式版の呼び出し回数の上限が合わない場合や、読み取りだけで足りる場合の代替になります。書き込みや Code Connect が必要なら公式版を選びます。

## まとめ

- Figma 公式の MCP サーバーが登場し、リモート版（`https://mcp.figma.com/mcp`）が推奨になりました
- API キーは不要で、OAuth 認証で接続します。呼び出し回数の上限はシートによって異なります
- フレーム単位のリンクを渡すのが基本です。Code Connect と組み合わせると、既存コンポーネントを活かした実装になります

新しい MCP 記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。

## 参考リンク

- [Figma MCP server - Figma Developer Docs](https://developers.figma.com/docs/figma-mcp-server/)
- [リモートサーバーのインストール - Figma Developer Docs](https://developers.figma.com/docs/figma-mcp-server/remote-server-installation/)
- [Tools and prompts - Figma Developer Docs](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/)
- [Guide to the Figma MCP server - Figma Help Center](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server)
- [Figma コネクタ - Claude](https://claude.com/connectors/figma)
- [Framelink Figma MCP - GitHub](https://github.com/GLips/Figma-Context-MCP)

次回は、Slack のメッセージ検索や投稿ができる「[Slack MCPサーバー](./mcp-server-tutorial-07-slack)」を解説します。
