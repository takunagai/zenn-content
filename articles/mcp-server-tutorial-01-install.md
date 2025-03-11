---
title: "【MCPのトリセツ #1】MCPの概要と導入方法"
emoji: "🐸"
type: "tech"
topics: ["mcp", "claude", "windsurf", "ai", "生成ai", "ai駆動開発"]
published: true
---

このシリーズでは、Claude や Windsurf Casade などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックを、初心者にもわかりやすく解説していきます。様々なMCPサーバーを活用して、AIとの協働作業をより豊かなものにしていきましょう！

### シリーズ目次

1. 👉 [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. [Slack MCPサーバー：チームコミュニケーションを強化](./mcp-server-tutorial-07-slack)
8. [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
9. [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)
11. [ウェブの情報を取得するMCPの使い分け (Fetch、Firecrawl、Markdownify)](./mcp-server-tutorial-reference-web-mcp)

---

## ✨ MCPとは？ 超シンプル解説

**MCP（Model Context Protocol）** は、AIにさまざまな機能拡張を安全に追加するための仕組みです。簡単に言えば「AIができること」を増やすためのプロトコルです。

従来のAIは：

- インターネット検索はできても、自分のPC内のファイルは読めない
- 画像生成はできても、手元の写真を編集できない

MCPを使うと：

- 指定したフォルダのファイルを読み書きできる
- 特定のアプリやWebサービスを操作できる
- 他にもいろいろな拡張機能を追加できる

つまり、MCPはAIとあなたのPCの間の「通訳」のような役割をしてくれるのです。

## どんなMCPサーバーが使える？ (MCPサーバーの一覧リスト)

- [modelcontextprotocol/servers - Github](https://github.com/modelcontextprotocol/servers?tab=readme-ov-file) 公式リポジトリ
- [Open-Source MCP servers - Glama](https://glama.ai/mcp/servers)
- [OpenTools.com](https://opentools.com/registry/) - 様々なAIツールを集めたディレクトリサイト
- [PulseMCP.com](https://pulsemcp.com/) - MCPサーバー専門のディレクトリ
- [Smithery.ai](https://smithery.ai/) - MCPサーバーのキュレーションサイト

---

## ✨ MCPサーバーの導入方法

## mcp-installer: 会話で指示してインストール

「MCPサーバーを導入したいけど難しそう...」
「コマンドラインやJSON設定ファイルの編集が苦手...」

そんな悩みを解決するのが [anaisbetts/mcp-installer - Github](https://github.com/anaisbetts/mcp-installer) 。このMCPサーバーは、他のMCPサーバーをインストールするためのメタツールです。

Claude Desktop など MCP が使えるサービスのチャットで `○○をインストールして` と指示するだけで、様々なMCPサーバーを簡単に導入できるようになります。mcp-installer の導入方法と使い方を解説します。

⚠️注意1：うまくいかない場合もあります。その場合は、マニュアルを参照しながらの手動設定が必要となります。

⚠️注意2：APIキーやアクセストークンなど、外に漏れてはいけない情報をチャットに入力しないようにしてください。必要な場合は、仮の値を入力して進めるか、`APIキーは後で手動で設定するのでダミー値を入力して` と指示し、後で手動で設定します。

### mcp-installer の特徴

- **会話で指示**: `MCPサーバー xxx をインストールして` と会話で指示するだけ
- **設定ファイル自動編集**: JSON設定ファイルを自動的に正しく更新
- **引数や環境変数も指定可能**: 複雑な設定も会話で指定できる
- **技術的知識不要**: コマンドラインやJSON形式の知識がなくても利用可能
- **多数のMCPサーバーに対応**: npm または PyPI でホストされているほとんどのMCPサーバーに対応

### 基本的な使用例

mcp-installer をセットアップすると、以下のようなプロンプトでMCPサーバーをインストールできます

```
MCPサーバー mcp-pandoc をインストールして
```

引数が必要な場合：

```
MCPサーバー @modelcontextprotocol/server-filesystem をインストールして。引数には['/users/username/desktop']を使用します
```

```
MCPサーバー @modelcontextprotocol/server-filesystem をインストールして。引数には['/Users/username/Documents', '/Users/username/Projects', '--read-only', '/Users/username/Reference']を使用します
```

APIキーやアクセストークンが必要な場合：

```
MCPサーバー @modelcontextprotocol/server-github をインストールして。環境変数 GITHUB_PERSONAL_ACCESS_TOKEN は後で手動で設定するのでダミー値を入力して
```

ローカルのものをインストールする場合：

```
MCPサーバーで /users/username/code/mcp-youtube をインストールして
```

このように、さまざまなタイプのMCPサーバーを会話だけで簡単にインストールできます。

## 🛠️ mcp-installer のインストール方法

### 前準備：uv のインストール

mcp-installer を使用するには、まず前提条件として「uv」というツールが必要です。uvは Python のバージョン管理やパッケージ管理を行うツールで、mcp-installer はこれを利用します。

### uvのインストール（macOS）

```bash
brew install uv
```

インストールが成功したか確認するには：

```bash
uv --version
```

バージョン番号が表示されればOKです。

> 💡 **メモ**: すでに pyenv や rye などの類似ツールを導入済みの場合は、それらを使うこともできますが、ここでは説明を省略します (私はこれを機に uv に乗り換えました)。

### mcp-installer のセットアップ

※ Claude Desktop での説明をします。Windsurf なら、`~/.codeium/windsurf/mcp_config.json` に同様の設定を追加してください。

mcp-installer をセットアップするには、Claude Desktop の設定ファイルを編集します。

### 1. 設定ファイルを開く

設定ファイルの場所は以下の通りです：

- **macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `C:\Users\ユーザー名\AppData\Roaming\Claude\claude_desktop_config.json`

Claude Desktop アプリからも上記の設定ファイルを開くことができます。
(メニュー: Claude > Settings… で、Developer タブを選び "Edit Config" ボタンを押す)

### 2. 設定内容の追加

ファイルを開いたら、以下の内容を追加します（既存のMCPサーバー設定がある場合は、mcpServers オブジェクト内に追加）：

※ 正確なパッケージ名を指定すること。例えば、"@anaisbetts/mcp-installer" を "mcp-installer" と省略するとエラーになります。

```json
{
  "globalShortcut": "",
  "mcpServers": {
    "mcp-installer": {
      "command": "npx",
      "args": [
        "@anaisbetts/mcp-installer"
      ]
    }
  }
}
```

この設定により、Claude AI から mcp-installer を呼び出せるようになります。

### 3. アプリを再起動し、動作確認

設定ファイルを保存したら、Claude Desktop アプリを再起動して設定を反映させます。
mcp-installerが正しく設定されたか確認するには、Claude に以下のように話しかけてみます。

```
今、MCP "@anaisbetts/mcp-installer" は使える状態になってる？
```

Claudeがmcp-installerの使い方について説明してくれれば、正しく設定されています！

## 📝 まとめ

mcp-installer は、MCPサーバーの導入を劇的に簡単にするメタツールです。コマンドラインやJSON設定ファイルの編集に不慣れな方でも、会話ベースでさまざまなMCPサーバーを簡単に導入できるようになります。

このツールを活用することで、MCPの世界への敷居がグッと下がり、AIの能力拡張をより身近に感じられるでしょう。ぜひmcp-installerを使って、様々なMCPサーバーを導入してみてください！

新しいMCP記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。フォローとツッコミ、お待ちしています！

## 📚 参考リンク

- [Anthropic MCP公式ドキュメント](https://docs.anthropic.com/ja/docs/agents-and-tools/mcp)
- [anaisbetts/mcp-installer - GitHub](https://github.com/anaisbetts/mcp-installer)
- [uv - GitHub](https://github.com/astral-sh/uv)

次回の記事では、ローカルファイルをAIから扱えるようにできる超絶便利な「[Filesystem MCP Server](./mcp-server-tutorial-02-filesystem)」について解説します。お楽しみに！
