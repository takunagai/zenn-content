---
title: "【MCPのトリセツ #12】Blender MCP Server: 会話で Blender を操作し3Dモデルを作成"
emoji: "🐸"
type: "tech"
topics: ["mcp", "windsurf", "ai", "生成ai", "blender"]
published: true
---

## 💡 MCPの始め方シリーズについて

Claude などの AI を強化する「MCP（Model Context Protocol）」の導入方法と活用テクニックのシリーズ。今回は、Blender MCPサーバーの導入方法と活用テクニックを紹介します。AIとの会話で3Dモデリングソフトウェア [Blender](https://www.blender.org/) を操作できるようになります！

![画像](/images/mcp-server-tutorial-12-blender/01.jpg)

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
12. 👉 [Blender MCP Server: AIでBlenderを操作](./mcp-server-tutorial-12-blender)
13. [ウェブの情報を取得するMCPの使い分け (Fetch、Firecrawl、Markdownify)](./mcp-server-tutorial-reference-web-mcp)

---

[blender-mcp - GitHub](https://github.com/ahujasid/blender-mcp)

## 🚀 Blender MCP Server でできること

Blender MCP Serverは、Blenderの操作を AI から行えるようにする MCPサーバーです。

- 双方向通信：Claude と Blender をソケットベースのサーバーで接続
- オブジェクト操作：3Dオブジェクトの作成、修正、削除
- マテリアル制御：マテリアルや色の適用と変更
- シーン検査：現在の Blender シーンの詳細情報取得
- コード実行：Blender で Pythonコードを実行

## 👨‍💻 Blender MCP Server プロンプトのサンプル

レンダリング前にカメラ位置調整のプロンプトを実行すると良いです。  
**※未検証のものも含まれています。うまくいかなかったらすみません💦**

### 基本的なオブジェクト操作

```
Blender で立方体を作成して。位置を(1, 1, 1)に移動して
```

```
選択中のオブジェクトを2倍に拡大して
```

### マテリアル操作

```
新しいマテリアルを作成して、赤色を適用して
```

```
選択中のオブジェクトのマテリアルを半透明にして
```

### シーン管理

```
現在のシーンにあるオブジェクトの一覧を表示して
```

```
カメラの位置を調整して、すべてのオブジェクトが見えるようにして
```

### 高度なモデリング

```
球体を作成して、ボーンアニメーション用のリグを設定して
```

```
選択中のメッシュにサブディビジョンサーフェスモディファイアを追加して
```

### レンダリング設定

```
Cyclesレンダラーに切り替えて、サンプル数を128に設定して
```

```
現在のビューをレンダリングして、PNG形式で保存して
```

### アニメーション

```
キューブを作成して、1秒間で360度回転するアニメーションを設定して
```

```
選択中のオブジェクトに、バウンドするアニメーションを追加して
```

### スクリプト実行

```
選択中のオブジェクトの頂点数を数えるスクリプトを実行して
```

```
すべてのオブジェクトの原点を中心に移動するスクリプトを実行して
```

### 高度なプロンプト例

#### 複雑なモデリング

```
低ポリゴンの木のモデルを作成して、葉のパーティクルシステムを設定して
```

#### マテリアルノードの設定

```
ノードベースのマテリアルを作成して、金属的な質感を表現して
```

#### アニメーションの組み合わせ

```
キャラクターモデルを読み込んで、歩行サイクルアニメーションを設定して
```

#### シーン最適化

```
シーン内のポリゴン数を最適化して、パフォーマンスを改善して
```

#### プロシージャルモデリング

```
プロシージャルな地形を生成して、テクスチャを自動的にマッピングして
```

### デモ動画の初めのプロンプト

[orange.aiさん - X](https://x.com/oran_ge/status/1899599891564999051)

```
Make a scene in Blender where there's a dragon standing next to a pot of gold. Make it isometric and make sure the lighting is playful and poilshed. Use the right materials. It should be worthy of posting on Dribbble!
```

---

## 🛠️ インストールと設定

Blender MCP Serverは、Python製のMCPサーバーです。インストールは非常に簡単です。
uv がインストール済みなのが前提です (インストール方法はシリーズ初回を参照)。

### Claude Desktop の設定

Claude Desktopの設定ファイル（claude_desktop_config.json）に以下を追加します：

```json
"mcpServers": {
  "blender": {
    "command": "uvx",
    "args": ["blender-mcp"]
  }
}
```

### Blender の設定

### Blender アドオンのインストール

1. [リポジトリ](https://github.com/ahujasid/blender-mcp?tab=readme-ov-file) から addon.py ファイルをダウンロード
2. Blender を起動
3. 編集 > プリファレンス > アドオン に移動
4. 右上のドロップダウンメニューから、"ディスクからインストール…" をクリックし、Addon.py ファイルを選択  
   → ~/Library/Application Support/Blender/X.X/scripts/addons/addon.py にインストールされる
5. 「Blender MCP」にチェックが付いていることを確認 (もし付いていないならアドオンを有効に)
6. 再起動

### Blender アドオンの起動

1. Blender を起動し、サイドバーを表示（メインビュー右上端にある `<` ボタン ※）
   ※ わからないくらいちっちゃいので注意
2. 「BlenderMCP」タブをクリックしパネルを開く
3. "Start MCP Server" ボタンをクリックし起動 (ポート番号を 9876 以外に変えたいなら変更)
4. この状態で Claude から会話で操作できるようになった！

---

## 📝 まとめ

Blender MCP Serverは、AIとBlenderを連携させる強力なツールです。以下のようなメリットが得られます：

- AIとの会話だけでBlenderを操作可能
- 複雑な3Dモデリング作業を自然言語で指示
- Pythonスクリプトの実行も可能
- 既存のBlenderワークフローに統合可能
- 他のMCPサーバーと組み合わせて使用可能

Blender MCPを導入することで、3Dモデリングをしたことがない人でも簡単な3Dモデルを手軽に作成できます。ぜひ導入して、AIとの新しい3Dモデリングワークフローを体験してみてください！

## 📚 参考リンク

- [Blender MCP GitHub リポジトリ](https://github.com/ahujasid/blender-mcp)
- [Blender Python API ドキュメント](https://docs.blender.org/api/current/index.html)
