---
title: "VSCode: PHP_CodeSniffer を使った WordPress コーディング規約の導入手順"
emoji: "🐸"
type: "tech"
topics: ["php", "wordpress", "開発環境", "vscode"]
published: true
---

WordPress の開発において、PHPコードを一定の規約に従わせることは、コードの品質と保守性を向上させます。そのために、`PHP_CodeSniffer` を導入し、コードのチェック・整形を自動化するのがおすすめです。この記事では、VSCode での導入方法を解説します。

## 1. PHP と Composer のインストール

PHP が入っていることを確認  
`php -v`

入っていなければ、PHPをインストール  
`brew install php`

Composer が入ってないことを確認  
`composer --version`

入っていなければ、Composer をインストール  
`brew install composer`

Composer がグローバルインストールで使っているディレクトリのパスを確認しとく  
`composer global config home`  
→ "/Users/ユーザー名/.composer"

## 2. PHP_CodeSniffer のインストール

Composer を使ってグローバルに PHP_CodeSniffer をインストールします。

phpcs と phpcbf をグローバルにインストール。  
`composer global require "squizlabs/php_codesniffer=*"`

インストール後、実行ファイルは次の場所に保存されます。

- `/Users/ユーザー名/.composer/vendor/squizlabs/php_codesniffer/bin/phpcs`
- `/Users/ユーザー名/.composer/vendor/squizlabs/php_codesniffer/bin/phpcbf`

上記の実行ファイル(シンボリックリンク)にPATHを通す。  
必要なら .zprofile を整理(並び順調整やコメント入れ)しとく。  
`echo 'export PATH=$HOME/.composer/vendor/bin:$PATH' >> ~/.zprofile`

設定を反映。  
`source ~/.zprofile`

PHP_CodeSniffer 動作確認  
`phpcs --version`  
`phpcbf --version`

PHP CodeSniffer の実行コマンドは以下の通り

- チェック: `phpcs チェックするファイルへのパス`
- 整形: `phpcbf  チェックするファイルへのパス`

## 3. WordPress コーディング規約の追加

WordPress用の規約をインストールします。

インストールされているコード規約の一覧を表示  
`phpcs -i`  
→ "MySource, PEAR, PSR1, PSR2, PSR12, Squiz and Zend” (現時点でWordPressのコーディング規約は入ってない)

規約関連ファイルの保存場所:  
~/.composer/vendor/squizlabs/php_codesniffer/src/Standards/

WordPress コーディング規約をインストール  
`composer global require wp-coding-standards/wpcs`  
→ "Do you trust…?" と聞いてくるので `y`

インストールされたか確認  
`phpcs -i`  
→ "MySource, PEAR, PSR1, PSR2, PSR12, Squiz, Zend, Modernize, NormalizedArrays, Universal, PHPCSUtils, WordPress, WordPress-Core, WordPress-Docs and WordPress-Extra"

規約関連ファイルの保存場所:  
~/.composer/vendor/wp-coding-standards/wpcs

WordPress コーディング規約をデフォルトとして設定 (これはプロジェクト設定するので不要だが一応)  
`phpcs --config-set default_standard WordPress`

PHP CodeSniffer の設定ファイルで “WordPress” が指定されてることを確認(※後でカスタマイズする)。  
~/.composer/vendor/squizlabs/php_codesniffer/CodeSniffer.conf

## 4. VSCode での設定

### プロジェクト独自のルール設定（カスタムルールの適用）

ルールを変更する理由は、"Inline comments must end in full-stops, exclamation marks, or question marksphpcs (インラインコメントは、フルストップ、感嘆符、または疑問符で終わらなければならない。)” エラーを無視させたいから。

まずは、カスタムルールファイル phpcs.xml (WordPress コーディング規約を参照、これを上書き) を作成し、ドキュメントルートに保存。これで特定のエラーを無視して自動整形・チェックが行える。

```phpcs.xml
<?xml version="1.0"?>
<ruleset name="Custom WordPress">
    <description>WordPress Coding Standards をカスタマイズしたもの</description>
    <rule ref="WordPress"/>
    <rule ref="Squiz.Commenting.InlineComment.InvalidEndChar">
        <severity>0</severity>
    </rule>
</ruleset>
```

### 機能拡張のインストール

機能拡張 [PHP Sniffer & Beautifier](https://marketplace.visualstudio.com/items?itemName=ValeryanM.vscode-phpsab) をインストール。

この機能拡張の設定を確認。Sniffer Mode が “onSave” になっているのを確認。

(これは不要かも↓)
Executable Path CBF に phpcbf コマンドのパス、Executable Path に phpcs コマンドのパスを設定。  setting.json で、この機能拡張の設定は `"phpsab.executablePathCS”` と phpsab が頭に付いたもの

### 機能拡張の設定ファイル

自動フォーマットをオンにし、フォーマッターを "valeryanm.vscode-phpsab” (PHP Sniffer & Beautifier 機能拡張の識別子) に指定する。  
そして、コーディング規約に上で追加した `phpcs.xml` を指定する。  
すでに設定済みのものがある場合は値を上書き変更(ちなみに自分の場合は、フォーマッターに "bmewburn.vscode-intelephense-client” (PHP Intelephense) がセットされていた)。

```setting.json
{
  "[php]": {
    "editor.defaultFormatter": "valeryanm.vscode-phpsab",
    "editor.formatOnSave": true
  },
  "phpsab.fixerArguments": [
    "--standard=phpcs.xml"
  ]
}
```

コーディング規約エラーがある PHPファイルを保存し、適用されればOKです。

## 参考記事

- [squizlabs/PHP_CodeSniffer - Github](https://github.com/squizlabs/PHP_CodeSniffer)
- [荒廃したWordPressのPHPコードをPHP_CodeSnifferでキレイにしたい #phpcs - Qiita](https://qiita.com/shin_m/items/a52530b40e1b868bd438)
- [VSCode WordPressのコーディングスタンダードで自動整形する - H.I. Art Works](https://tech.hi-works.com/webcreative/1064)
