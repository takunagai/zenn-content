---
title: "Next.js x MicroCMS で構築した Jamstack サイトを Netlify にデプロイする手順"
emoji: "🐸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nextjs", "microcms", "netlify", "frontend"]
published: true
---

## やりたいこと

* Netlify でデプロイ。無料プランを使用
* app router や next/images が動く
* Github にプッシュ → Netlify が自動再ビルド
* MicroCMS で記事の追加/更新/削除 → Netlify が自動再ビルド
* 独自ドメインの設定 (今回は Cloudfrare Domains との接続)

## Netlify についてのメモ

無料版はリージョン制限があるため、速さを求めるなら有料プランにする必要がある

* 無料 Starter プラン
  * 月100GBの帯域、300分のビルド時間
  * １ユーザーのみ、同時ビルドは１つのみ
  * Functions region が "US East (Ohio) - us-east-2" 固定
  * 東京リージョンの CDNはあるが、有料プランでないと選べない
* 参考
  * [料金とプラン | Netlify](https://www.netlify.com/pricing/)
  * [Netlify の CDN のリスト | Netlify Support Forums](https://answers.netlify.com/t/is-there-a-list-of-where-netlifys-cdn-pops-are-located/855/2)
  * [Netlify が遅いので Vercel に移行した | Lambdar](https://www.lambdar.me/archives/migrating-to-vercel-from-netlify-due-to-performance-issues/)

## 手順

以下、ドメイン名を hoge.com として進めるので、適宜自分のものに変換してください。  
ローカルの開発環境で Next.js サイトが正常に動いてる状態からの解説となります。

### 1. ローカル環境で正常にビルドできることを確認

開発環境 (`npm run dev`) で動いていても、ビルドするとエラーが出る場合があるので、事前にチェックしておく。

* `npm run build`
* `npm run start` → ブラウザで動作確認

### 2. プロジェクトを Github にプッシュ

今後、共同開発や引き継ぎがあった場合に備え、今回は Organization 下で管理する。共同開発や引き継ぎを想定していないなら個人のリポジトリでOK。

* Github で Organization を作成
* Public でプロジェクト用のリポジトリを新規作成し、ローカルのプロジェクトとの関連付け＆アップロード (手順は割愛)

### Github と Netlify の接続

Netlify の管理画面で Netlify とリポジトリの関連付けを行う

* 環境変数をセット
  * Site configuration > Environment variablesで
  * .env.local に書いてる設定 (MicroCMS API Key など) をセット
* 管理画面でデプロイを行う
  * → デプロイは成功するが、next/images で設定した画像が表示されない。これは Netlify CLI (V5) を適用することで解決できる。→ 次項を参照

### Netlify CLI (v5) のセットアップ

Netlify CLI (v5)のセットアップで、app router や next/image が動くようになります。サポートされているのは Next.js 13.5 以降、Node 18+。  
※ 未満なら v4 を使うしかないので、これ以降の内容は役に立ちません。

[Get started with Netlify CLI - Netlify Docs](https://docs.netlify.com/cli/get-started/)

:::details Netlyfy CLI とは
Netlify のコマンドラインインターフェース (CLI) を使うと、コマンドラインから直接継続的デプロイメントを構成できます。Netlify CLI を使用すると、他のユーザーと共有できるローカル開発サーバーを実行したり、ローカル ビルドとプラグインを実行したり、サイトをデプロイしたりできます。
:::

Netlyfy CLI は、グローバルとローカル(= プロジェクト)、いずれにもインストールできます。今回はローカルにインストールします。

Netlify CLI のインストール (プロジェクトルートで)
`npm install netlify-cli --save-dev`

バージョンやコマンドを確認
`npx netlify`

トークンを取得
`netlify login`
→ 表示されるリンクをクリックし、ブラウザで認証。認証が成功すると "You are now logged into your Netlify account!" と出ます。

初期化
`npx netlify init`
→ 以下のプロントが出るので答えていきます。

プロンプト："What would you like to do?"
→ "Connect this directory to an existing Netlify site" を選択

プロンプト："How do you want to link this folder to a site?"
→ "Use current git remote origin /" を選択し、リポジトリ選択

"Directory Linked" と表示され、リンクされたらOK！
プロジェクトを確認すると .netlify/state.json が生成され、`siteId` が設定されているのが確認できます。

ステータスの確認
`npx netlify status`

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

### 再びデプロイ

再度、Netlify でデプロイを行います。next/images の Image コンポーネントで設定した画像が表示されているのが確認できたらOKです。

### デプロイの自動化

#### Github リポジトリにプッシュしたら自動デプロイ

この時点で、すでにできてます。

#### MicroCMS で記事を更新したら自動デプロイ

1. Netlify で、MicroCMS 用のフックを作成＆コピー
    * Netlify 管理画面の Build & deploy > Continuous deployment の Build hooks で 作成。生成されたフックをコピーしておく
2. MicroCMS 側で設定
    * MicroCMS の API設定 > Webhook > Netlify で 1 でコピーしたものをセット。反映させたい API 全てに同じものを同様に適用します。
    * [コンテンツのWebhookを設定 - microCMS](https://document.microcms.io/manual/webhook-setting)
3. デプロイ通知の設定
    * Netlify 管理画面の Site configuration › Notifications、Site configuration › Emails and webhooks

### 独自ドメインの設定

今回は、Cloudfrare domains で独自ドメインを取得済みという前提での解説です。

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
"https://" からの URL でアクセスし、サイトにアクセスできたら OK。

### おわりに

Vercel は Next.js を使う上ではこの上ないサーバーですが、無料枠の制限が厳しめだったり、有料にしたときの料金体系が直感的にわかりづらいです。無料枠に優しく料金もわかりやすい Netlify で、今は app router や next/images が手間なく普通に使えるようになったのは大変ありがたいです。
