---
title: "デフォルトはArcブラウザ、ターミナルからは Chrome を実現する「Finicky」とその設定 (Mac)"
emoji: "🐸"
type: "tech"
topics: ["claude-code", "dev-environment", "ai-agent", "terminal", "vscode"]
published: true
---

Claude Code から [Claude in Chrome](https://www.claude.com/chrome) を使ってブラウザ操作（開発ツールの使用、クリック、フォーム入力など）ができるようになり、「ターミナルから開くリンクは、普段のデフォルトブラウザ（Arc）ではなく Chrome で開きたい！」という状況になりました。Sidebar API 非対応の Arc では Claude in Chrome のブラウザ拡張が使えないためです。  

条件により開くブラウザを切り替えてくれる Mac アプリ [Finicky (v4)](https://github.com/johnste/finicky)を使うことで、以下が実現できます。

- 通常は Arc をデフォルトブラウザにする  
- ターミナル (ターミナルエミュレーターや IDE/エディタのターミナルも含む) から開いた URL は Chrome で開く

## 導入手順

### 1. Finicky をインストール

[GitHub](https://github.com/johnste/finicky/releases/) から最新版をダウンロードしてインストールします。(v4.3のアルファ版が出てるが、この記事では v4.2.2 を使用)

### 2. Finicky の設定ファイル（.finicky.js）を作成し設置

以下を設置し、ホームディレクトリ直下 (`~/.finicky.js`) に保存します。

```javascript:.finicky.js
export default {
  defaultBrowser: {
    name: "company.thebrowser.Browser", // Arc
    appType: "bundleId"
  },
  options: {
    checkForUpdates: true,
    // logRequests: true, // ログ出力 (開いたアプリなども確認できる)
  },
  handlers: [
    {
      // まず全部拾って browser で分岐
      match: () => true,

      browser: (url, { opener }) => {
        // http/https 以外は Arc
        if (!["http:", "https:"].includes(url.protocol)) {
          return { name: "company.thebrowser.Browser", appType: "bundleId" };
        }

        // Terminal の `open` などで opener 情報が取れず空になるケース対策
        const openerEmpty =
          !opener || !(opener.bundleId || opener.name || opener.path);

        if (openerEmpty) {
          return { name: "com.google.Chrome", appType: "bundleId" };
        }

        const bid = opener.bundleId;

        const devOpeners = new Set([
          // ターミナルとそのエミュレータ
          "com.apple.Terminal",        // Terminal.app
          "com.googlecode.iterm2",     // iTerm2
          "org.alacritty",             // Alacritty
          "co.zeit.hyper",             // Hyper
          "com.github.wez.wezterm",    // WezTerm

          // IDE・エディタ
          "com.microsoft.VSCode",          // Visual Studio Code
          "com.todesktop.230313mzl4w4u92", // Cursor（固定ID）
          "com.exafunction.windsurf",      // Windsurf
          "com.google.antigravity",        // Antigravity
          "dev.zed.Zed",                   // Zed
          "com.cmux.app",                  // cmux
        ]);

        // JetBrains IDEs
        const isJetBrains = bid.startsWith("com.jetbrains.");

        // Cursor は配布形態で bundleId が変わることがあるので prefix でも拾う
        const isCursorLike = bid.startsWith("com.todesktop.");

        const shouldUseChrome = devOpeners.has(bid) || isJetBrains || isCursorLike;

        return shouldUseChrome
          ? { name: "com.google.Chrome", appType: "bundleId" }
          : { name: "company.thebrowser.Browser", appType: "bundleId" };
      },
    },
  ],
};
```

### 3. Mac のシステム設定で Finicky をデフォルトブラウザにする

システム設定 > デスクトップとDock > デフォルトのWebブラウザ で、"Finicky.app" を選択します。  
(Finicky.app は自動起動してくれるので、ログイン時に開く設定など不要でした。)

### 4. Finicky を起動

デフォルトブラウザが開く状況で Arc が起動、ターミナルから `open https://google.com` を実行すると Chrome が起動することが確認できたら OK！

- - -

ここからは、仕様には関係がないため、興味がある人だけ読んでください。

[Finicky 公式ドキュメント](https://github.com/johnste/finicky/wiki/Configuration-(v4))を参考に設定しても、なかなかうまくいきませんでした。ターミナルからURLを開くと、どうしても Arc で開いてしまう…。Finicky v4 の `opener` の扱いを正しく理解することが解決のカギでした。

## 成功のポイント（要点だけ）

### 1. `opener` は「必ず入ってくる」と思わない
Finicky v4 では、`browser(url, options)` の `options.opener` が

- `null`
- もしくは `{ bundleId: "", name: "", path: "" }` のような 空オブジェクト

で渡ってくるケースがあります（特に `open` コマンド経由）。

これを考慮しないと、条件分岐がすべて外れて  
デフォルトブラウザ（Arc）に落ちる原因になります。

### 2. `match` で絞らず、`browser` を関数にして分岐する
Finicky v4 では `browser` を関数で書けます。

- `match` は常に `true`
- 実際の分岐は `browser` の中で行う

この形にすると、起動元判定が安定します。

### 3. 起動元が「空」なら Chrome に倒す
ターミナル起点で Chrome を使いたい場合、

- `opener` が空 = ターミナル `open` 経由の可能性が高い

と割り切り、Chrome に倒すのが一番事故が少ないようです。

### 4. JetBrains / Cursor は prefix 判定が安全
JetBrains 系や Cursor は、エディションや配布形態で  
`bundleId` が変わることがあるため、

- `startsWith("com.jetbrains.")`
- `startsWith("com.todesktop.")`

のような prefix 判定を併用します。
