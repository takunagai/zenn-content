---
title: "【MCPのトリセツ #5】GitHub MCPサーバー： AIでリポジトリを管理"
emoji: "🐸"
type: "tech"
topics: ["mcp", "github", "claude", "windsurf", "ai", "生成ai", "ai駆動開発"]
published: true
---

## 💡 MCPの始め方シリーズについて

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。今回は、GitHub MCPサーバーの導入方法と活用例を解説します。開発ワークフローをぐっと効率化しましょう！

### シリーズ目次

1. [MCPの概要と導入方法](./mcp-server-tutorial-01-install)
2. [Filesystem MCP Server： AIでローカルファイルを扱う](./mcp-server-tutorial-02-filesystem)
3. [YouTube MCPサーバー：動画の内容を取得](./mcp-server-tutorial-03-youtube)
4. [mcp-pandoc： AIでドキュメント形式を変換](./mcp-server-tutorial-04-pandoc)
5. 👉 [GitHub MCPサーバー： AIでリポジトリを管理](./mcp-server-tutorial-05-github)
6. [Figma MCP：デザインとコードを効率的に連携](./mcp-server-tutorial-06-figma)
7. [Slack MCPサーバー：チームコミュニケーションを強化](./mcp-server-tutorial-07-slack)
8. [Firecrawl MCP：スクレイピングでウェブ情報を取得・分析](./mcp-server-tutorial-08-firecrawl)
9. [Markdownify MCP Server: WebページやPDFをMarkdown文書化](./mcp-server-tutorial-09-markdownfy)
10. [Raindrop.io MCP Server: 便利なブックマークサービスをAIから使う](./mcp-server-tutorial-10-raindropio)
11. [ウェブの情報を取得するMCPの使い分け (Fetch、Firecrawl、Markdownify)](./mcp-server-tutorial-reference-web-mcp)

---

## ✨ GitHub MCPサーバー

「AI に直接リポジトリを作成して、作業中のプロジェクト一式をプッシュできたら便利なのに...」
「AI補助で作成したイシューのリストを、直接 Github のイシューにアップできたらいいのに...」

GitHub MCPサーバーで、これらのことができるようになります。このツールを使えば、AIとGitHubを直接連携させ、Gitコマンドを覚える必要がなくリポジトリの操作やコード管理を会話ベースで行えるようになります。個人アクセストークンのスコープを制限して安全に利用できるので安心です。

## 🛠️ セットアップ手順

GitHub MCPサーバーをセットアップするには、以下の2つのステップが必要です。

### 1. GitHub 個人アクセストークンの取得

まず、GitHub API にアクセスするための個人アクセストークン(PAT)を取得します。

1. [GitHub](https://github.com/)にログイン
2. 右上のプロフィールアイコン → Settings をクリック
3. 左側メニューを下にスクロールし、Developer settings をクリック
4. Personal access tokens → Tokens (classic) を選択
5. Generate new token (classic) をクリック
6. トークンの設定：
   - **名前**: `My GitHub Token (MCP用)` など、わかりやすい名前を付ける
   - **権限**: リポジトリ関連の権限を選択（最小限の権限を推奨）
   - **有効期限**: 必要に応じて設定（セキュリティの観点から30日程度を推奨）
7. Generate token ボタンをクリック
8. 表示されたトークンをコピーして安全な場所に保存（再表示されないため注意）

> ⚠️ **セキュリティ注意**: トークンは絶対に公開しないでください。GitHubリポジトリや公開の設定ファイルに含めることは避けましょう。トークンが漏洩した場合は、すぐに無効化して新しいものを発行してください。

### 2. MCPサーバーの設定

#### 方法1：手動で設定

##### Claude Desktop の場合

1. Claude Desktop の設定ファイル（`~/Library/Application Support/Claude/claude_desktop_config.json`）を開く
2. 以下の設定を追加（複数のMCPサーバーがある場合は "mcpServers" オブジェクト内に追加）。

  ```json
  {
    "mcpServers": {
      "github": {
        "command": "npx",
        "args": [
          "-y",
          "@modelcontextprotocol/server-github"
        ],
        "env": {
          "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_your_token_here"
        }
      }
    }
  }
  ```

3. `ghp_your_token_here` の部分を、先ほど取得したGitHubトークンに置き換えます。

##### Windsurf の場合

1. `~/.codeium/windsurf/mcp_config.json` を開く（なければ作成）
2. 同様の設定を追加
3. `ghp_your_token_here` の部分を、先ほど取得したGitHubトークンに置き換えます。

#### 方法2： mcp-installer を使う場合

**※これ、実際に試してません。うまくいかなかったら申し訳ありません。**

前回または[シリーズ記事#0](./MCPサーバー%2000%20簡単に導入する手順%20\(mcp-installer\).md)でmcp-installerをセットアップ済みであれば、Claudeに以下のように指示できます：

```
MCPサーバー @modelcontextprotocol/server-github をインストールして。環境変数 GITHUB_PERSONAL_ACCESS_TOKEN は 'your_token_here' で設定します
```

その後、`ghp_your_token_here` の部分を、先ほど取得したGitHubトークンに置き換えます。

### 3. MCPサーバーの再起動

インストールが完了したら、Claude Desktop を再起動(自動的にMCPサーバーが起動します)。
手動で起動する場合は、ターミナルで以下のコマンドを実行してMCPサーバーを起動します。

```bash
npx -y @modelcontextprotocol/server-github
```

---

## 👨‍💻 使用例 (プロンプト)

GitHub MCPサーバーの使い方を紹介します。  
(※後半は実際に試していなかったり、実用的なものが含まれている可能性があります。)

### 1. リポジトリの基本操作

#### 1-1. リポジトリの作成 (名前や説明、公開設定なども指定可)

```
Github でリポジトリ "test-repo" を新規作成して
```

```
「my-portfolio」という名前の新しいリポジトリを作成して。説明は「個人ポートフォリオサイト」、プライベートリポジトリで、READMEファイルも初期化してください
```

#### 1-2. README.md ファイルの作成・編集

```
そのリポジトリに README.md を新規作成して。内容は、Webアプリケーションのテンプレートを説明する基本的な内容でOKです。
```

```
README.mdに、プロジェクトの特徴、インストール手順、使用例のセクションを追加してください
```

#### 1-3. ブランチの作成と管理

```
dev ブランチを作成して
```

```
「feature/user-authentication」というブランチを main から作成して
```

#### 1-4. ファイルの追加とコミット

```
index.html を作成し、シンプルなHTMLテンプレートを記述して。その後、コミットして
```

```
src/components/Login.js ファイルを作成し、Reactのログインコンポーネントを実装して。コミットメッセージは「ログインフォームコンポーネントの追加」としてください
```

#### 1-5. プルリクエストの作成と管理

```
「feature/user-authentication」ブランチから「main」ブランチへのプルリクエストを作成してください。タイトルは「ユーザー認証機能の実装」、説明には実装した機能の概要とテスト方法を含めてください
```

```
プルリクエスト #5 にコメントを追加して：「CIテストが通過したので、レビューをお願いします」
```

### 2. リポジトリの情報取得と分析

#### 2-1. リポジトリの検索と取得

```
キーワード "javascript tutorial" で、スターが5000以上のリポジトリを検索して
```

```
「react state management」に関する、最近更新された人気のリポジトリを5つ探して、それぞれの特徴を簡潔に説明してください
```

#### 2-2. コードの検索と分析

```
GitHubで「react custom hooks」というキーワードで検索し、よく使われているカスタムフックの実装パターンを3つ見つけて説明してください
```

```
リポジトリ「myusername/my-project」内で、セキュリティの脆弱性が疑われるコードパターンを検索して報告してください
```

#### 2-3. イシューの検索と分析

```
「react performance」に関するイシューを人気のリポジトリから検索し、よく議論されている最適化テクニックをまとめてください
```

```
リポジトリ「facebook/react」で、最近クローズされた重要なバグ修正のイシューを5つ見つけて、それぞれどのような問題が解決されたのか説明してください
```

### 3. 実用的な開発支援例

#### 3-1. リポジトリを分析しコードレビュー

```
私のリポジトリ「myusername/my-project」のコードをレビューして、改善点を指摘してください。特にパフォーマンスとセキュリティの観点から見てください。
```

```
リポジトリ「myusername/e-commerce-app」のフロントエンドコードを分析し、アクセシビリティの観点から改善すべき点を具体的に指摘してください
```

#### 3-2. ブランチの差分を分析し、プルリクエストを作成

```
「feature-login」ブランチの変更内容を分析して、mainブランチへのマージのためのPull Requestを作成してください。PR内容には実装した機能の概要と、テスト方法を含めてください。
```

```
「bugfix/payment-processing」ブランチと「main」ブランチの差分を分析し、どのような問題が修正されたのか詳細に説明してください
```

#### 3-3. イシュー管理の効率化

```
リポジトリ「myusername/my-project」の未解決Issueをリストアップして、優先度ごとに分類してください。また、比較的簡単に解決できそうなIssueを3つピックアップしてください。
```

```
このバグについて新しいイシューを作成してください：「ログイン後にユーザープロフィールが正しく表示されない。特にFirefoxブラウザで再現する問題です。」タグには「bug」「frontend」「priority-high」を付けてください
```

#### 3-4. プロジェクト初期化の自動化

```
React+TypeScriptのプロジェクトテンプレートを新しいリポジトリ「my-new-app」に作成してください。ESLint、Prettier、Jestのセットアップも含めてください。
```

```
「next-blog-starter」という名前の新しいリポジトリを作成し、Next.js、Tailwind CSS、Contentfulを使ったブログサイトの基本構造をセットアップしてください
```

#### 3-5. ドキュメント生成と管理

```
リポジトリ「myusername/my-library」のコードから、APIドキュメントを生成して、docs/フォルダに保存してください。各関数の使用例も追加してください。
```

```
リポジトリ「myusername/cli-tool」のコマンドラインインターフェースの使い方を説明するマークダウン形式のドキュメントを作成し、examples.mdとして保存してください。主要なコマンドとオプションの使用例を含めてください
```

### 4. チーム開発とコラボレーション

#### 4-1. コントリビューターガイドラインの作成

```
オープンソースプロジェクト「myusername/community-app」用のCONTRIBUTING.mdファイルを作成してください。コードスタイル、プルリクエストのプロセス、イシューの報告方法を詳しく説明してください
```

#### 4-2. リリースノートの生成

```
リポジトリ「myusername/my-app」の最新バージョンv2.0.0のリリースノートを作成してください。前回のリリースv1.5.0からの主要な変更点、新機能、バグ修正を含めてください
```

#### 4-3. プロジェクト管理の効率化

```
リポジトリ「myusername/team-project」のマイルストーンを作成してください。「Q2 Release」という名前で、説明には「第2四半期のリリース目標機能」とし、期限は2025年6月30日に設定してください
```

### 5. 高度な活用例

#### 5-1. コードの移行と変換

```
リポジトリ「myusername/javascript-app」のコードをTypeScriptに移行するための計画を立ててください。移行の手順、必要な変更点、推奨ツールを詳細に説明してください
```

#### 5-2. セキュリティ監査

```
リポジトリ「myusername/e-commerce-api」のセキュリティ監査を行い、潜在的な脆弱性、ベストプラクティスからの逸脱、改善すべき点を詳細にレポートしてください
```

#### 5-3. パフォーマンス最適化

```
リポジトリ「myusername/web-app」のフロントエンドコードを分析し、パフォーマンスを最適化するための具体的な提案をしてください。特にレンダリングパフォーマンスとバンドルサイズに注目してください
```

#### 5-4. 複数リポジトリの統合分析

```
私の「myusername/backend-api」と「myusername/frontend-client」リポジトリを分析し、APIインターフェースの一貫性と統合ポイントをチェックしてください。不整合がある場合は指摘してください
```

---

## ⚠️ GitHub MCPサーバーを安全に使用するための注意点 (セキュリティと制限事項)

1. **最小権限の原則**: 個人アクセストークンには必要最小限の権限だけを付与しましょう。
2. **有効期限の設定**: トークンには適切な有効期限を設定しましょう。
3. **機密リポジトリの扱い**: 極めて重要なプロジェクトでは、追加のセキュリティ対策を検討しましょう。
4. **トークンの管理**: トークンは安全に保管し、定期的に更新しましょう。

---

## 🔄 他のMCPサーバーとの組み合わせ

他のMCPサーバーと組み合わせることでさらに便利になります。

### Filesystem MCP + GitHub MCP

ローカルのコードを分析し、GitHubにプッシュするといった連携が可能です：

```
1. /Users/yourname/Projects/my-app/ のコードを分析して、改善点を提案してください。
2. 改善したコードをGitHubリポジトリ「myusername/my-app」にプッシュしてください。
```

### YouTube MCP + GitHub MCP

YouTube チュートリアル動画の内容を GitHubリポジトリに実装。

```
1. このYouTubeチュートリアル（https://www.youtube.com/watch?v=xxxxx）の内容を分析してください。
2. チュートリアルで説明されている機能を実装したコードを新しい GitHubリポジトリに作成してください。
```

## 📝 まとめ

GitHub MCPサーバーを導入することで、AIとGitHubの連携が劇的に向上します。リポジトリの作成・管理、コードレビュー、Issue管理など、様々な作業を自然言語での会話を通じて行えるようになります。

開発者がリポジトリを管理する場合や、GitHubの操作に不慣れな場合に助けとなるでしょう。セキュリティに注意しながら、ぜひGitHub MCPサーバーを活用してみてください！

新しいMCP記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。フォローとツッコミ、お待ちしています！

## 📚 参考リンク

- [GitHub MCPサーバー公式リポジトリ](https://github.com/modelcontextprotocol/servers/tree/main/src/github)
- [GitHub API ドキュメント](https://docs.github.com/en/rest)
- [GitHub 個人アクセストークンの管理](https://github.com/settings/tokens)
- [MCP初心者ガイド動画](https://www.youtube.com/watch?v=Y_kaQmhGmZk)

次回の記事では、デザインとコードを効率的に連携させる「[Figma MCP](./mcp-server-tutorial-06-figma)」について解説します。お楽しみに！
