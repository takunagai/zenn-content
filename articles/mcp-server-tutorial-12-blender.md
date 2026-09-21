---
title: "【MCPのトリセツ #12】Blender MCP Server: 会話で Blender を操作し3Dモデルを作成"
emoji: "🐸"
type: "tech"
topics: ["mcp", "blender", "claude", "claudecode", "codex"]
published: true
---

Claude や Codex などの AI に外部ツールをつなぐ「MCP（Model Context Protocol）」の導入方法と使い方を解説するシリーズです。今回は、3D 制作ソフト [Blender](https://www.blender.org/) を AI との会話で操作できるようにする MCP サーバーを取り上げます。

:::message
**更新日: 2026-09-22**（初版: 2025-03-12）

紹介していた `blender-mcp` は「MCP for Blender」（パッケージ名 `mcp-for-blender`）に改名されました。アドオンの導入がコマンド 1 つになり、素材サイトや 3D 生成 AI との連携、安全モードが加わっています。Blender の開発元による公式の MCP サーバーも登場したので、あわせて紹介します。
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
12. **Blender MCP Server: 会話で Blender を操作し3Dモデルを作成（この記事）**
13. [Perplexity MCP Server: Perplexity ならではの検索をAIとの会話で実行](./mcp-server-tutorial-13-perplexity)
14. [国土交通省がMCPサーバーを公開：AI時代のオープンデータ活用45選](./mcp-server-tutorial-14-milt-data)

資料: [ウェブ情報を取得するMCPの比較 (Fetch、Firecrawl、Markdownify、Perplexity)](./mcp-server-tutorial-reference-web-mcp)

---

## MCP for Blender でできること

[MCP for Blender](https://github.com/ahujasid/blender-mcp)（旧 blender-mcp）は、Blender 内で動くアドオンと MCP サーバーの 2 つで構成されています。アドオンが Blender の中に待ち受け口を作り、MCP サーバーが AI からの指示をそこへ中継します。Blender の開発元とは無関係の、コミュニティ製のツールです。

| 機能 | 内容 |
|---|---|
| オブジェクト操作 | 3D オブジェクトの作成、変更、削除 |
| マテリアル制御 | マテリアルと色の適用、変更 |
| シーンの確認 | 現在のシーンの詳細情報を取得 |
| コード実行 | Blender 内で Python コードを実行 |
| 素材とモデルの取得 | Poly Haven、Sketchfab、Poly Pizza の素材の検索とダウンロード |
| 3D モデルの生成 | Hyper3D Rodin、Hunyuan3D による AI 生成 |

Blender のような「起動中のアプリとつながり続ける」用途は、スキルや CLI では置き換えにくく、MCP サーバーが向いている典型例です。

## セットアップ手順

必要なものは、Blender 3.0 以降と uv です。uv は `pip install uv` ではなく、公式のインストーラーか Homebrew で入れます（[シリーズ #1](./mcp-server-tutorial-01-install) を参照）。

### MCP サーバーを登録する

Claude Desktop は、設定ファイルに次を追加して再起動します。

```json
{
  "mcpServers": {
    "blender": {
      "command": "uvx",
      "args": ["mcp-for-blender"]
    }
  }
}
```

Claude Code と Codex は 1 行で登録できます。

```bash
claude mcp add blender uvx mcp-for-blender
```

```bash
codex mcp add blender -- uvx mcp-for-blender
```

旧版の記事のとおり `uvx blender-mcp` で設定済みの場合も、そのまま動き続けます。設定を書き換える必要はありません。

MCP サーバーは同時に 1 つだけ動かします。Claude Desktop と Claude Code の両方から同時に接続すると、うまく動きません。

### Blender のアドオンを入れる

旧版では `addon.py` を手動でダウンロードしていましたが、現在はコマンドで Blender のアドオンフォルダにコピーされます。

```bash
uvx mcp-for-blender install-addon
```

そのあと Blender を開き、「編集 > プリファレンス > アドオン」で「Interface: MCP for Blender」を有効にします。

### 接続する

1. Blender の 3D ビューポートで `N` キーを押し、サイドバーを表示する
2. 「MCP for Blender」タブを開く
3. 使いたい連携（Poly Haven など）のチェックボックスを入れる
4. 「Start MCP Server」を押す

この状態で、AI から Blender を操作できます。

### uvx が見つからないとき

Claude Desktop のように GUI から起動するアプリは、ターミナルの PATH を引き継ぎません。`spawn uvx ENOENT` というエラーが出たら、ターミナルで `which uvx` を実行し、表示されたフルパス（`/opt/homebrew/bin/uvx` など）を `command` に書きます。

## 安全モード

既定では、AI は Blender の中で任意の Python コードを実行できます。ファイルの読み書きや外部プログラムの実行もできてしまうので、環境変数 `BLENDER_MCP_SAFE_MODE=1` を設定して、実行前にスクリプトを検査する安全モードを有効にしておくのが無難です。

```json
{
  "mcpServers": {
    "blender": {
      "command": "uvx",
      "args": ["mcp-for-blender"],
      "env": {
        "BLENDER_MCP_SAFE_MODE": "1"
      }
    }
  }
}
```

README も「使う前に必ず作業を保存すること」と警告しています。AI の操作でシーンが壊れることはあるので、試す前に `.blend` ファイルを保存しておきます。

## プロンプトのサンプル

### 基本の操作

```text
Blender で立方体を作成して、位置を (1, 1, 1) に移動して
```

```text
現在のシーンにあるオブジェクトの一覧を見せて
```

```text
新しいマテリアルを作成して赤色にし、選択中のオブジェクトに適用して
```

```text
カメラの位置を調整して、すべてのオブジェクトが画面に収まるようにして
```

### モデリングとレンダリング

```text
選択中のメッシュにサブディビジョンサーフェスのモディファイアを追加して
```

```text
ローポリの木のモデルを作って。幹は茶色、葉は緑のマテリアルにして
```

```text
キューブを作成して、1 秒間で 360 度回転するアニメーションを設定して
```

```text
レンダラーを Cycles に切り替えてサンプル数を 128 にし、現在のカメラからレンダリングして PNG で保存して
```

### 素材サイトと生成 AI を使う

```text
Poly Haven の HDRI、テクスチャ、岩や植物のモデルを使って、ビーチの雰囲気のシーンを作って
```

```text
Hyper3D で庭に置くノームの置物の 3D モデルを生成して、シーンの中央に配置して
```

Poly Haven の素材はすべて CC0 で、API キーも要りません。Sketchfab、Poly Pizza、Hyper3D、Hunyuan3D は、それぞれの API キーを Blender のサイドバーで設定します。Poly Haven の素材は解像度が 1 段階上がるごとにファイルサイズが約 4 倍になり、ダウンロード中は Blender の操作が止まります。カメラの近くに置くものでなければ、1k か 2k を指定するのが現実的です。

### 対話を重ねて仕上げる

一度の指示で完成させようとせず、結果を見ながら修正を重ねるのがコツです。公開直後に話題になった [orange.ai さんのデモ](https://x.com/oran_ge/status/1899599891564999051)も、次のような指示から始めて、20 回近いやり取りで仕上げています。

```text
Blender で、金の壺の隣にドラゴンが立っているシーンを作って。アイソメトリックの構図で、遊び心のあるライティングにして
```

```text
一度に全部やるのではなく、段階を踏もう。まず壁、次に壁の松明、最後に細部の順で追加して
```

```text
松明が壁にめり込んでいるので、シーンの内側に出して。炎は発光するマテリアルにして
```

「めり込んでいる」「近すぎる」のように、見えている問題をそのまま伝えると修正が通りやすくなります。

## Blender 公式の MCP サーバー

Blender の開発元も、実験的な取り組みの場である Blender Lab で [MCP Server](https://www.blender.org/lab/mcp-server/) を公開しています。Blender の Python API を自然言語で扱うこと、ドキュメントを引きやすくすること、複雑なシーンの構成を調べて理解することを目的にした軽量なサーバーで、こちらも専用のアドオンを Blender に入れて使います。

素材サイトや生成 AI との連携まで含めて手早く試すなら MCP for Blender、開発元のツールで Python API を中心に使いたいなら公式版、という分け方になります。どちらも AI が生成したコードを Blender 内で実行する仕組みなので、作業の保存と、信頼できない指示を読み込ませない注意は共通です。

## まとめ

- `blender-mcp` は MCP for Blender（`mcp-for-blender`）に改名されました。旧設定はそのまま動きます
- アドオンは `uvx mcp-for-blender install-addon` で入ります。Poly Haven などの素材や 3D 生成 AI とも連携できます
- 任意の Python コードを実行できるので、安全モードを有効にし、作業を保存してから使います

新しい MCP 記事の更新は X [@nagataku_ai](https://x.com/nagataku_ai) でお知らせします。

## 参考リンク

- [MCP for Blender（ahujasid/blender-mcp）- GitHub](https://github.com/ahujasid/blender-mcp)
- [MCP Server - Blender Lab](https://www.blender.org/lab/mcp-server/)
- [Blender Python API ドキュメント](https://docs.blender.org/api/current/index.html)
- [Poly Haven](https://polyhaven.com/)

次回は「[Perplexity MCP Server](./mcp-server-tutorial-13-perplexity)」を解説します。
