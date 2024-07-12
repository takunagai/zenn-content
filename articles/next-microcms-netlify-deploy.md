---
title: "Next.js x MicroCMS で構築した Jamstack サイトを Netlify にデプロイする手順"
emoji: "🐸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "microcms", "netlify", "frontend"]
published: true
---

## やりたいこと

* Netlify でデプロイ
* app router や next/images が動く
* Github にプッシュ → Netlify が自動再ビルド
* MicroCMS で記事の追加/更新/削除 → Netlify が自動再ビルド
* 独自ドメインの設定 (今回は Cloudfrare Domains との接続)

## 手順

ドメイン名を hoge.com としているので、適宜自分のものに変換してください。

### ローカル環境で正常にビルドできることを確認

* `npm run build`
* `npm run start`

### プロジェクトを Github にプッシュ

今後、共同開発や引き継ぎがあった場合に備え、今回は Organization 下で管理する。

* Github で Organization を作成
* Public でプロジェクト用のリポジトリを新規作成し、ローカルの該当プロジェクトとの関連付け＆アップロード (手順は割愛)

### Github と Netlify の接続

* Netlify でそのリポジトリとの関連付けを行う
* 環境変数をセット (Site configuration > Environment variablesで。MicroCMS API Key など .env.local に書いてるやつ)
* 管理画面でデプロイを行う
  → 成功するが、next/images で設定した画像が表示されない。これは Netlify CLI (V5) を適用することで解決できる。→ 事項を参照

### Netlify CLI (v5) のセットアップ

Netlify CLI (v5)のセットアップで、app router や next/image が動くようになる。
サポートは Next.js 13.5 以降、 Node 18+ (未満なら v4 を使うしかない)。

[Get started with Netlify CLI - Netlify Docs](https://docs.netlify.com/cli/get-started/)

Netlyfy CLI とは
> Netlify のコマンドライン インターフェース (CLI)を使用すると、コマンドラインから直接継続的デプロイメントを構成できます。Netlify CLI を使用すると、他のユーザーと共有できるローカル開発サーバーを実行したり、ローカル ビルドとプラグインを実行したり、サイトをデプロイしたりできます。

グローバルとローカル、いずれにもインストールできるが、今回はローカル (プロジェクトルート) にインストールする

Netlify CLI のインストール (プロジェクトルートで)
`npm install netlify-cli --save-dev`

バージョンやコマンドを確認
`netlify`

トークンを取得
`netlify login`
表示されるリンクをクリックでブラウザでの認証画面に。
認証が成功すると "You are now logged into your Netlify account!"

初期化
`netlify init`

プロンプト："What would you like to do?"
　→  "Connect this directory to an existing Netlify site" を選択

プロンプト："How do you want to link this folder to a site?"
　→ "Use current git remote origin /" を選択し、リポジトリ選択

 "Directory Linked" と表示され、リンクされたらOK！

ステータスの確認
`netlify status`

```
──────────────────────┐
 Current Netlify User │
──────────────────────┘
Email: ****@****.***
Teams:
  ****: Owner Developer Reviewer
────────────────────┐
 Netlify Site Info  │
────────────────────┘
Current site: ****
Admin URL:    https://app.netlify.com/sites/****
Site URL:     https://****.netlify.app
Site Id:      ****-****-****
```

Netlify でデプロイを行い、成功することを確認。

### デプロイの自動化

#### Github リポジトリにプッシュしたら自動デプロイ

すでにできている

#### MicroCMS で記事を更新したら自動デプロイ

1. MicroCMS 用のフックを作成＆コピー
    * Netlify の Build & deploy > Continuous deployment の Build hooks で 作成
2. MicroCMS 側の設定
    * MicroCMS の API設定 > Webhook > Netlify で 1 でコピーしたものをセット。反映させたい API 全てに同じものを適用する
    * [コンテンツのWebhookを設定 - microCMS](https://document.microcms.io/manual/webhook-setting)
3. デプロイ通知の設定
    * Site configuration › Notifications、Site configuration › Emails and webhooks

### 独自ドメインの設定

今回は Cloudfrare domains で独自ドメインを取得済みとした説明。

* Netlify の Domain management で、取得済みのドメインを入力し追加
* 取得済みのドメインなので "Add domain" ボタンを押す
* "Awaiting External DNS" リンクをクリックで手順説明 (A) が見れる

#### DNS 設定

##### DNS configuration

上の (A) の手順説明を読む

> 推奨：ALIAS、ANAME、または平坦化されたCNAMEレコードをapex-loadbalancer.netlify.comに向ける
あなたのDNSプロバイダがALIAS、ANAME、または平坦化されたCNAMEレコードをサポートしている場合は、フォールバックオプションより回復力のあるこの推奨設定を使用してください。
> apex-loadbalancer.netlify.comのロードバランサーを指すhoge.comのALIAS、ANAME、または平坦化されたCNAMEレコードを作成します。
> `hoge.com ALIAS apex-loadbalancer.netlify.com`

Cloudfrare domains は、フラット化 CNAME レコード (CNAME Flattening) をサポートしているのでこの方法で。
(フラット化 CNAME レコードをサポートしていないレジストラの場合は、同説明の Fallback の項を参照)
`hoge.com ALIAS apex-loadbalancer.netlify.com` を設定に使う。

* Cloudfrare Domains の DNS > Record で "CNAME" で Name を "@" (ドメインルート)、Content を "apex-loadbalancer.netlify.com" と設定する

:::details フラット化 CNAME レコードとは (by ChatGPT)
> CNAME Flattening とは、特にDNS（Domain Name System）において、CNAME（Canonical Name）レコードをルートドメイン（例：example.com）に設定できるようにする技術です。
> CNAMEレコードはサブドメイン（例：www.example.com）にのみ設定でき、ルートドメインにはAレコード(ドメイン名をIPv4アドレスに変換)やAAAAレコードドメイン名をIPv6アドレスに変換を設定する必要があります。しかし、CNAME Flatteningを使うことで、この制約を取り除き、ルートドメインに対してもCNAMEレコードを設定できるようになります。
> メリット：ルートドメインに対してもCNAMEを設定できるためDNS設定がシンプルになる。CNAMEのターゲットが変更されても自動的にIPアドレスが更新される
:::

##### Netlify DNSを使用する

> 最高のパフォーマンスを得るために、Netlifyの高性能なマネージドDNSサービスをご利用ください。Netlify DNS は自動的にすべてのホスト名に最適なレコードを作成し、また、当社のブランチサブドメイン機能を簡単に使用することができます。

hoge.com に Netlify DNS を設定する。
最下部のリンク "Set up Netlify DNS for hoge.com →" をクリック

##### Set up Netlify DNS for your domain

Netlify の Domains の Name servers で４つの アドレスが確認できる。

* dns1.pXX.nsone.net (※ 環境によって異なるかも)
* dns2.pXX.nsone.net
* dns3.pXX.nsone.net
* dns4.pXX.nsone.net

これをコピーし、Cloudfrare Domains の DNS > Record で "NS" でこの４つを追加。
(該当ドメイン画面の右サイトバーの "Update DNS configuration" から設定。Name は @ でそのドメインのルートとなる)
※伝播に時間がかかる(24時間以内)ので、設定後すぐには反映されない。少し待つ必要がある。

### HTTPS 化

Netlify 管理画面 Domain management > HTTPS で、"Verify DNS configuration" ボタンを押すだけ。設※※伝播に時間がかかる(24時間以内)ので、設定後すぐには反映されない。少し待つ必要がある。

### おわりに

Netlify でも app router や next/images が手間なく普通に使えるようになったのは大変ありがたい。
