---
title: "至高の Markdown エディタ「MarkEdit」 (Mac版のみ)"
emoji: "🐸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["markdown", "開発環境"]
published: true
---

MarkEdit は、シンプル＆軽快さと使い勝手を重視した Markdown エディタ。オープンソースなので無料で使えます。私が気に入っている点や改善してほしい点、設定のコツなどを紹介します。

入手はこちらから  
[MarkEdit-app/MarkEdit - Github](https://github.com/MarkEdit-app/MarkEdit)

## 気に入っている点

### 圧倒的な軽さ

MarkEdit は驚くほど軽快です。テキストエディットよりも軽いと感じるくらいサクサク動きます。「早いと軽いは正義」と感じる方にはぴったりです。

### コードブロックのシンタックスハイライトと折りたたみ機能

文書中にプログラミングコードを埋め込むなら、シンタックスハイライトは欠かせません。MarkEdit にはデフォルトで20種類以上のテーマが用意されており、好きな見た目にカスタマイズ可能です。私のお気に入りは [Night Owl](https://github.com/haishanh/night-owl.vim) という落ち着いたテーマです。

また、行ナンバー横の三角マークをクリックすることでコード中のタグや関数などを折りたためます。今は不要な箇所を一時非表示にすることで、見通しよく編集できます。

![シンタックスハイライト](/images/markedit-markdown-editor/theme.png)

### コードブロックの入力補助

コードブロックを簡単に挿入できるショートカット機能も充実しています。`コマンド + シフト + C` で選択したコードを一発でコードブロックに変換できるので、作業が捗ります。

![コードブロックのショートカットキー](/images/markedit-markdown-editor/code-block.png)

### 入力補助とショートカットキー

クォーテーションを入力すると、自動で閉じクォーテーションが追加されます。また、文字列を選択した状態でクォーテーションを打つと、その文字列をクォーテーションで囲ってくれます。リンクの入力も簡単で、URL をコピーした状態でリンクさせたい文字列を選択し、｀k｀ボタンを押すだけです。

### 複数のカーソル操作が可能（マルチキャレット）

複数の箇所を同時に編集できる「マルチキャレット機能」が便利です。コマンドキーを押しながらクリックすることで、複数の編集ポイントを同時に選択して作業できます。プログラミング用エディタでは当たり前の機能ですが、Markdown エディタでは珍しいのではないでしょうか。

![マルチキャレット編集](/images/markedit-markdown-editor/multi-carret.png)

### プレビューがなくても読みやすい

MarkEdit にはプレビュー表示機能がありませんが、テーマがスタイル付け(見出しを太字で大きくして色付けなど)してくれるので、十分に可読性を確保できます。Markdown ではどうしても見づらいテーブルに限り、プレビューボタンでプレビューができると気が利いています。

![テーブルのプレビュー](/images/markedit-markdown-editor/table-preview.png)

### Pandoc を使った書き出し機能

MarkEdit 自体には PDF や HTML、Word文書への書き出し機能はありませんが、メニューから Pandoc 経由で簡単に変換できます。「文書変換は Pandoc 以上のものがないから」という潔い理由でそうなっているそうです。  
(メモ: 設定ファイルは、ユーザー/Library/Containers/app.cyan.markedit/Data/Documents/pandoc.yaml)

![Pandoc](/images/markedit-markdown-editor/pandoc.png)

### バージョン管理機能

macOS の Time Machine に対応しており、今開いているファイルを過去のバージョンに戻すことができます。うっかりミスは起こるもの。安心です。

![Mac TimeMachine](/images/markedit-markdown-editor/time-machine.png)

### Macネイティブな設計

macOS Sonoma で追加された「インライン予測」や macOS Sequoia で追加された「ライティングツール」が、MarkEdit 上で動作します。

## 改善してほしい点

同じ文字列が文書内でハイライト表示される機能は便利ですが、すべての一致箇所を選択するのが手間です。 `Cmd + E` で選択した文字列が検索欄にセットされ、一致箇所がハイライトされっぱなしになるのでマルチキャレット選択しやすくはなりますが、ポチポチとひとつづづ選択するのは面倒です(できるけど自分がやり方を知らないだけかも)。

## おすすめの設定

「Edit > Spelling and Grammar > Collect Spelling Automatically」をオフにすることをおすすめします。デフォルト(ON)のままだと、半角英字入力中に意図しないスペル修正をされ、とても煩わしいです。

## 自分の執筆環境

私は今まで、階層管理が可能なノートアプリとして「Evernote」「Simple Note」「メモ.app」「Notion」などを使用してきました。しかし今はデータとして Markdown を素で扱い、階層を OS のフォルダ構造で実装している「iA Writer」をメインに使っています。そして、編集をこの軽快な「MarkEdit」で行います。

この記事も MarkEdit で執筆し、iA Writer で管理しています。そして、Zenn 記事投稿用として作成したフォルダを VSCode のプロジェクトとしています。記事の仕上げは VSCode で Zenn 拡張機能のサポートを受けながら仕上げました。

![文書管理と執筆のフロー](/images/markedit-markdown-editor/writing-flow.svg)

Markdown なので、今後自分が蓄積した文書群を AI の学習データとしたいとなった時も、親和性がバッチリでしょう。

プレーンなテキストファイルである Markdown のシンプルさと柔軟性が、この流れるような作業フローを可能にします。しかも、裏では OS が自動でバージョン履歴を取ってくれています。

## カスタマイズ

2024年10月4日時点ではまだ少ないですが、機能拡張があり開発もできます。便利なものが増えてきたら別記事として書こうかと思います。  
[» Customization · MarkEdit-app/MarkEdit Wiki](https://github.com/MarkEdit-app/MarkEdit/wiki/Customization)

## まとめ

MarkEdit はシンプルかつ強力な Markdown エディタです。Mac ユーザーで Markdown 形式で執筆されている方で使ったことがない方は、ぜひ試してみてください。

「道具は、使う人によって初めて価値が決まる。」

[» 公式の推しポイント](https://github.com/MarkEdit-app/MarkEdit/wiki/Why-MarkEdit)
