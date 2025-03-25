---
title: "Windsurf で公式の VSCode 拡張機能を使えるようにする方法"
emoji: "🐸"
type: "tech"
topics: ["windsurf", "ai", "生成ai", "ai駆動開発", "開発環境"]
published: true
---

Windsurf ではデフォルトで Open-VSX から拡張機能を取得します。Microsoft の制限により、公式の Microsoft VSCode Marketplace にある拡張機能が入手できない場合があり、また、入手できるものでも最新バージョンがない場合があります。
以下の手順で公式の VSCode 拡張機能を入手できるように切り替えることができます。

1. 右上のアイコンドロップメニューから Windsurf Settings を開く
2. Marketplace Extension Gallery Service URL を変更
    - 元：`https://open-vsx.org/vscode/gallery`
    - 変更：`https://marketplace.visualstudio.com/_apis/public/gallery`
3. Marketplace Gallery Item URL を変更
    - 元：`https://open-vsx.org/vscode/item`
    - 変更：`https://marketplace.visualstudio.com/items`
4. 再起動。拡張機能パネルの検索欄下に "Using marketplace.visualstudio.com." と表示されたらOK！

これで公式の VSCode Marketplace の拡張機能が利用可能になります！  
ただし、Github Copilot など、Windsurf 側で使えなくしている機能拡張もあり、それらのインストールはできません。

元情報：[Codemium/Windsurf コミュニティ- Discord](https://discord.com/channels/1027685395649015980/1351572913332949107/1351572913332949107)
