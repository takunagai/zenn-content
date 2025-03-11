---
title: "Mac プレビューで Markdown 文書を表示する (QLMarkdown)"
emoji: "🐸"
type: "tech"
topics: ["markdown", "開発環境", "mac"]
published: true
---

## 概要

[sbarex/QLMarkdown - Github](https://github.com/sbarex/QLMarkdown)

Mac の「プレビュー」機能（スペースキーでファイルを素早くプレビューする機能）で Markdown ファイルをパッとプレビュー表示できるようにする方法を紹介します。ドキュメント管理が快適になる小技です！

![快適なMarkdownプレビュー表示](/images/mac-ql-markdown-preview/01.png)

## インストールと設定

1. インストール  
`brew install --cask qlmarkdown`

2. アプリケーションフォルダから、QLMarkdown.app を右クリックで開く。警告が出るので許可。

3. 再起動  
`qlmanage -r`  

4. Markdown ファイルをスペースで開くと、Github Flavored Markdown スタイルのプレビュー表示されることを確認

### スタイルの変更

1. 任意の作業フォルダで CSS を作成 (下記参照。ファイル名は任意)
2. QLMarkdown.app をクリックし開き、Theme > Import で 1 をインポート  
   (→ 以下の保存場所に複製保存される)
3. `qlmanage -r` で Quicklook を再起動
4. Markdown ファイルをスペースで開くと、自作のスタイルのプレビュー表示されることを確認

#### 保存場所

`~/Library/Group Containers/org.sbarex.qlmarkdown/Library/Application Support/themes/light-theme.css`

#### CSS

```light-theme.css
/* light-theme.css */
body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif;
  font-size: 14px;
  line-height: 1.4;
  color: #333333;
  background-color: #ffffff;
  margin: 0;
  padding: 30px;
}

a {
  color: #0366d6;
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}

h1, h2, h3, h4, h5, h6 {
  margin-top: 24px;
  margin-bottom: 16px;
  font-weight: 600;
  line-height: 1.25;
  color: #24292e;
}

h1 {
  font-size: 2em;
  border-bottom: 1px solid #eaecef;
  padding-bottom: 0.3em;
}

h2 {
  font-size: 1.5em;
  border-bottom: 1px solid #eaecef;
  padding-bottom: 0.3em;
}

code {
  font-family: "SFMono-Regular", Consolas, "Liberation Mono", Menlo, monospace;
  color: #ffffff;
  background-color: #f6f8fa;
  padding: 0.2em 0.4em;
  border-radius: 3px;
  font-size: 85%;
}

pre {
  background-color: #f6f8fa;
  border-radius: 3px;
  padding: 16px;
  overflow: auto;
}

pre code {
  background-color: transparent;
  padding: 0;
}

blockquote {
  margin: 0;
  padding: 0 1em;
  color: #6a737d;
  border-left: 0.25em solid #dfe2e5;
}

table {
  border-collapse: collapse;
  width: 100%;
  overflow: auto;
  margin-bottom: 16px;
}

table th, table td {
  padding: 6px 13px;
  border: 1px solid #dfe2e5;
  background-color: #ffffff;
}

table tr {
  background-color: #ffffff;
  border-top: 1px solid #c6cbd1;
}

table th {
  color: #ffffff;
  background-color: #666666;
}

table tr:nth-child(2n) td {
  background-color: #e7e7e7;
}

img {
  max-width: 100%;
}

ul, ol {
  padding-left: 2em;
}

hr {
  height: 0.25em;
  padding: 0;
  margin: 24px 0;
  background-color: #e1e4e8;
  border: 0;
}
```

## まとめ

Homebrew で簡単にインストールでき、カスタム CSS でスタイルを自由にカスタマイズできるため、自分の好みに合わせた快適な環境を簡単に構築できます。

Markdown を日常的に使用している方は、ぜひ導入してみてください。
