---
title: "【MCPのトリセツ #2】Filesystem MCP Server： AIでローカルファイルを扱う"
emoji: "🐸"
type: "tech"
topics: ["mcp", "github", "claude", "windsurf", "ai", "生成ai", "ai駆動開発"]
published: true
---

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。今回は、AIでローカルファイルを扱えるようになる超絶便利な「Filesystem MCP Server」の設定方法を解説します。

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. 👉 [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. [Slack MCPサーバー：チームコミュニケーションを強化](./mcp-server-tutorial-07-slack)
8. [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
9. [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)

---

## 🔍 Filesystem MCP Server の何が便利？

AIアシスタントを使っていて「ローカルのファイルを分析してくれたら便利なのにな…」と思ったことはありませんか？

セキュリティ上の理由から、Claude Desktop の LLM サーバーはローカルファイルにアクセスできません。しかし MCP を使うことで、安全にローカルファイルを扱うことが可能になります！

AIにローカルファイルへのアクセスを許可すると、こんなメリットがあります：

1. **プライバシー向上** - 機密文書をクラウドにアップロードせずに処理できる (※ただし、Claude などの LLMサーバーにはデータが送信されるので注意)
2. **作業効率アップ** - 「このファイルを分析して」と直接指示できる
3. **シームレスな作業** - 文書作成・編集・分析が一貫して行える
4. **一括処理** - 複数のファイルを一度に処理できる

例えば、ローカルPCにあるドキュメントフォルダで以下のようなことができるようになります。ドキュメント作業や開発作業が多い方なら、作業時間を大幅に短縮できるでしょう。

- ファイルやフォルダの新規作成/読み取り/更新/削除/移動/複製
- 複数ファイルを読み、AIに分析してもらう
- 複数ファイルをまとめて編集
- テキストファイルの分析
- ファイル情報の取得
- ファイル検索

## 🛠️ セットアップ手順：基本の3ステップ

私は mcp-installer でのインストールは失敗しました。なので、手動でインストールする手順を説明します。

### 1. 必要なツールをインストール

まずはターミナル（コマンドプロンプト）を開いて以下を実行します：

```bash
npm install -g @modelcontextprotocol/server-filesystem
```

> 💡 **ヒント**: Node.jsがインストールされていない場合は、[Node.jsの公式サイト](https://nodejs.org/)からダウンロードしてインストールしてください。

### 2. 設定ファイルを編集

使用するアプリに応じた設定ファイルを見つけて(存在しなければ新規作成して)編集します：

#### Claude Desktop

- **MacOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

#### 設定内容

"args" の3つ目の引数以降に、アクセスを許可したいディレクトリのパスを指定します。

`/Users/yourname/Documents`、`/Users/yourname/Projects` の部分は、実際にアクセスを許可したいディレクトリのパスに置き換えてください。カンマ区切りで連ねることで、複数のディレクトリを指定できます。

Windows の場合は、`C:\\Users\\あなたのユーザー名\\Desktop` などのように指定してください。

```json
{
  "mcpServers": {
    "file-system": {
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

### 3. アプリの再起動と動作確認

Claude Desktop（または設定したエディタ）を終了し、再起動します。
そして、ローカルファイルを操作する指示を試してみましょう。

```mcp
/Users/yourname/Desktop にあるファイルの一覧を取得しファイル名を列挙して
```

指定フォルダの内容に言及してくれたら成功です！

## ✨ 便利に使うテクニック

### 直感的な言葉でフォルダにアクセスできるようにする

例えば `/Users/yourname/Desktop にあるファイルの一覧を取得して` を `デスクトップフォルダにあるファイルの一覧を取得して` と指示できるようにします。

やり方は、カスタムインストラクションに対応表を追加するだけ。
Claude Desktop なら、右下のアカウントメニューから、Settings > Profile に移動し、"What personal preferences should Claude consider in responses?" に以下を追加します(※設定前に、下の"気をつけること"を必ず参照してください)。

```
ローカルのフォルダは以下とします。
- 指定がない場合:  '~/AI-Workspace'
- プロジェクトフォルダ: '~/Projects'
- ダウンロードフォルダ: '~/Downloads'
- デスクトップフォルダ: '~/Desktop'
- ドキュメントフォルダ: '~/Library/Mobile Documents/XXX/Documents'
- プレビューフォルダ: '~/Library/Mobile Documents/com~apple~Preview/Documents'
```

デフォルトの作業用フォルダ(上の設定例では `AI-Workspace`)を指定しておくことで、AIで生成した情報をとりあえずローカルに保存したい場合に、簡単な指示で作業用フォルダに保存でき便利です。

```
AIで生成した情報をローカルに保存して
```

## 気をつけること

- ⚠️必要最小限のフォルダ以外にはアクセス許可を付与しないでください。特にシステムディレクトリへのアクセスは絶対に禁止
- ⚠️上書きや削除などができるので、重要なディレクトリやファイルを操作する場合は十分注意してください。Gitバージョン管理で操作履歴を保存しておくことをおすすめします。
- 大量のファイルを一度に操作すると、応答時間が長くなるし、LLMが扱えるコンテキストを超過してうまく動作しなくなります。大量のファイルを含むフォルダ内の複数ファイルを操作したい場合は、作業ディレクトリにファイルを配置し、そこで作業するのがおすすめです。
- 「ファイル一覧を取得して」と指示した時に、完全に正確なファイル名が返ってこない場合があります。LLMがコンテキストにストックする際に加工をしていたりするためです。正確さを求める場合はその旨を指示してください。

---

## 🚀 プロンプトのサンプル

ブログ記事の作成支援(今回のシリーズはこれを使用):

```
ドキュメントフォルダ/AI/MCP にある "MCPサーバー 01" 〜 "MCPサーバー 07" の記事の内容や構成を確認して把握して
```

```
ここからあなたは技術ブログが得意なコピーライターです。
"MCPサーバー 01" 〜 "MCPサーバー 07" の全記事を以下の条件に従って統一感のあるものにリライトして。別の場所にバックアップは取ってあるので、編集内容を上書きしてください。

- MCP初心者向けの技術ブログ記事
- "MCPの始め方" というシリーズをこれら８記事で構成 (今後追加の可能性あり)
- 冒頭にシリーズ記事である旨の内容と目次を設置
- わかりやすく、テンポよく読める構成や文章に
```

データ分析レポート自動作成:

```
/Users/yourname/Projects/sales-data/ フォルダのCSVファイルを分析して、
売上傾向を調査し、グラフ入りのレポートを作成して
/Users/yourname/Documents/reports/sales-report.md に保存してください。
```

### 開発・コーディング関連

これらはエディタでやる方がいいが、一応メモ。

コードリファクタリング支援:

```
/Users/yourname/Projects/my-app/src/ のJavaScriptファイルを解析して、
重複コードやパフォーマンス問題を見つけ、修正案を示してください。
修正後のコードは optimized/ フォルダに保存してください。
```

プロジェクト初期化の自動化:

```
新しいReactプロジェクトのセットアップを手伝ってください：
1. /Users/yourname/Projects/new-react-app/ ディレクトリを作成
2. package.jsonを作成し、必要な依存関係を定義
3. 基本的なディレクトリ構造とファイルを作成
4. プロジェクトのREADME.mdファイルを生成
```

### 他のMCPサーバーとの連携

### Filesystem MCP Server + GitHub

GitHub の Issue や Pull Request を分析し、それに基づいてローカルコードを修正。

### Filesystem MCP Server + YouTube

YouTube動画の内容テキストを分析してローカルに保存したり、動画で紹介されたコードを実装したり。

## 📝 まとめ

Filesystem MCP Serverの導入により、Claude や AIエディタなどの AIアシスタントがローカルファイルにアクセスできるようになります。これにより、ドキュメント管理、コード分析、データ処理などの作業が大幅に効率化されます。

セキュリティに注意しながら、AIとの新しい協働スタイルを探ってみてください。MCPは単なるファイルアクセスの枠を超えて、AIとの共同作業の可能性を広げる技術です。

新しいMCP記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。フォローとツッコミ、お待ちしています！

## 📚 参考リンク

- [Anthropic MCP公式ドキュメント](https://www.anthropic.com/mcp)
- [Claude Desktop アプリ](https://claude.ai/desktop)
- [@modelcontextprotocol/server-filesystem パッケージ](https://www.npmjs.com/package/@modelcontextprotocol/server-filesystem)

次回の記事では、YouTubeの動画コンテンツをAIが理解できるようにする「YouTube MCPサーバー」について解説します。
