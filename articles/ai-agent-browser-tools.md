---
title: AI エージェントにブラウザを操作させる4ツールの選び方
emoji: 🐸
type: tech
topics:
  - claudecode
  - browser
  - mcp
  - playwright
  - chromedevtools
published: true
---

AI エージェントにブラウザを触らせたい。そんな時に候補に挙がるのが、**Claude in Chrome**、**chrome-devtools-mcp**、**agent-browser**、**Playwright MCP** 。どれも「AI 向けのブラウザ操作」ができるツールですが、それぞれ特徴があります。

銀の弾丸的なものは存在せず、用途によって最適解が入れ替わります。この記事ではまず用途別でおすすめのツールを挙げ、比較と解説はその後にします。どれを入れるか迷っている人にお役に立てれば幸いです。

:::message
数値・仕様は 2026-06-21 時点の調査に基づきます。この分野は動きが速いので、導入前に各公式の最新情報を確認してください。
:::

## 結論：用途から選ぶ

### 単体で選ぶなら

| 目的                                        | 選ぶツール                              |
| ----------------------------------------- | ---------------------------------- |
| Firefox / WebKit も含めたクロスブラウザ E2E・CI 回帰テスト | **Playwright MCP**（クロスブラウザは一択）     |
| Lighthouse 監査・メモリリーク・パフォーマンス診断            | **chrome-devtools-mcp**（診断機能が最も厚い） |
| ログイン必須の画面（SSO / OAuth / 2FA）の操作・動作確認      | **Claude in Chrome**（純正・権限 UI 付き）  |
| Chrome での AI 操作を軽量・高速に、トークンも節約したい         | **agent-browser**（CLI ファースト）       |

### 組み合わせるなら

単体で完結しない案件では、役割を分担させるのが現実解です。以下は一例です。

- **よくある構成**：操作は Claude in Chrome または agent-browser、品質ゲートは chrome-devtools-mcp、CI 回帰は Playwright MCP
- **Vercel / Next.js 中心の開発**：操作は agent-browser、診断は chrome-devtools-mcp
- **書いた直後に同じセッションで動作確認**：Claude in Chrome 単体、または chrome-devtools-mcp（auto-connect 起動）を併用

## 4ツールの比較（機能）

| 観点            | Claude in Chrome         | chrome-devtools-mcp    | agent-browser            | Playwright MCP                     |
| ------------- | ------------------------ | ---------------------- | ------------------------ | ---------------------------------- |
| 提供元           | Anthropic                | Google Chrome          | Vercel Labs              | Microsoft                          |
| 形態            | Chrome 拡張＋Claude Code 統合 | MCP（CDP 経由）            | CLI＋MCP（Rust・CDP 直結）     | MCP（Playwright 経由）                 |
| 主軸            | 操作（実ログイン）                | 診断                     | 操作（CLI 効率）               | 自動化・テスト                            |
| 既定のブラウザ       | 実 Chrome / Edge          | Chrome（auto-connect 可） | ヘッドレス／auto-connect／クラウド  | クリーン隔離                             |
| クロスブラウザ       | Chrome / Edge            | Chrome のみ              | Chrome 系のみ               | Chromium / Firefox / WebKit / Edge |
| Lighthouse 監査 | ✕                        | ◎                      | ✕（Web Vitals のみ）         | ✕                                  |
| ログイン継承        | ◎ 標準で共有                  | ◎ auto-connect 時       | ◯ auto-connect 時         | △ 都度再現                             |
| CI・無人実行       | ✕                        | △                      | ◎                        | ◎                                  |
| トークン効率        | 中                        | 中（--slim で改善）          | ◎ CLI 前提                 | 中                                  |
| 前提・料金         | 要 Anthropic 有料プラン（ベータ）   | 無料（OSS）                | 無料（OSS）                  | 無料（OSS）                            |
| 導入起点          | `claude --chrome`        | プラグイン / npx            | `npm i -g agent-browser` | MCP 登録                             |

機能の対応範囲と、安定性・実績は別の軸です。次に公開時期や Issue 数といった数字を見ます。

## 数字で見る規模と新しさ

| 指標 | Claude in Chrome | chrome-devtools-mcp | agent-browser | Playwright MCP |
|---|---|---|---|---|
| GitHub スター | —（拡張のため該当なし） | 44.1k | 36.6k | 34.2k |
| Open Issues | — | 88 | 531 | 7 |
| 初回公開 | 2025-08 | 2025-09 | 2026-01 | 2025-03 |
| MCP としての実績 | Claude Code に統合済 | 成熟 | 2026-06 追加 | 成熟 |

スター数はリリース時の話題性を映す指標で、実運用での信頼とは別物です。Open Issues も月数と開発速度に左右されるため、枯れた Playwright MCP の 7 件と新興の agent-browser の 531 件は単純比較できません。新しさと実績のどちらを優先するかは、本番要件しだいです。

## 各ツールの解説

各ツールで強い軸が異なります。用途に近いものを参照してください。

### Claude in Chrome

Anthropic 純正の Chrome 拡張で、Claude Code に統合されています。`claude --chrome` または `/chrome` で有効化すると、普段使っている Chrome / Edge のログイン状態を共有したまま、クリック・入力・データ抽出・GIF 記録などを任せられます。

- **強い軸**：実ログイン済みセッションをそのまま操作できる。SSO / OAuth / 2FA を突破し直す必要がない。グラフィカルな権限 UI で操作範囲を制御できる純正の安心感
- **弱い軸**：Lighthouse のような深い診断はできない。クロスブラウザは Chrome / Edge まで。利用には Pro / Max / Team / Enterprise の直接 Anthropic プランが必要で、現状ベータ
- **有用なシーン**：ログインが要る管理画面のバグを、認証し直さずに調べたい
- 導入：[公式ドキュメント](https://code.claude.com/docs/ja/chrome)

### chrome-devtools-mcp

Google Chrome チームが公式に出している MCP サーバーです。Chrome DevTools Protocol（CDP）経由で、Lighthouse 監査・ヒープスナップショット・パフォーマンストレースといった DevTools の診断機能をエージェントに直接渡します。

- **強い軸**：Lighthouse のフルスコア（パフォーマンス / アクセシビリティ / SEO / ベストプラクティス）を取得できる。ヒープスナップショットや perf trace でメモリ・速度も調べられる。Google Chrome チームの公式プロジェクト
- **弱い軸**：Chrome 専用でクロスブラウザは不可。クリックやフォーム入力といった操作系を主目的にすると、診断特化のぶん遠回りになる
- **有用なシーン**：書いたページの表示速度・a11y・メモリを、コードと同じセッションで診断したい
- 導入：[実践ガイド（拙稿）](https://zenn.dev/takna/articles/claude-code-chrome-devtools-mcp) / [GitHub](https://github.com/ChromeDevTools/chrome-devtools-mcp)

### agent-browser

Vercel Labs の、Rust 製ネイティブ CLI です。Pure Rust デーモンが CDP を直接叩く構成で、Node.js を介さず高速に動きます。CLI ファーストなのでトークン効率がよく、`agent-browser mcp` で MCP サーバーにもなります。Browserbase などのリモート・クラウド実行にも対応します。

- **強い軸**：CLI 前提でコンテキスト消費が小さい。Rust デーモンで軽量・高速。ローカルからクラウドまでポータブルに動き、Claude Code 以外のエージェントでも使える。設計が新しく、リモート実行を最初から織り込んでいる
- **弱い軸**：Chrome / Chromium 系のみで Firefox / WebKit は非対応（CDP 直結ゆえの構造的な制約）。Lighthouse は非搭載で Web Vitals の計測まで。MCP モードは 2026 年 6 月に追加されたばかりで、実運用の蓄積はこれから
- **有用なシーン**：Chrome での操作を軽量・高速に回したい。Vercel / Next.js の開発フローに組み込みたい
- 導入：[agent-browser.dev](https://agent-browser.dev/) / [GitHub](https://github.com/vercel-labs/agent-browser)

### Playwright MCP

Microsoft の Playwright をバックエンドにした MCP サーバーです。クリーンな隔離ブラウザを起動し、アクセシビリティツリーをもとに操作します。テストの自動生成にも対応します。

- **強い軸**：Chromium / Firefox / WebKit / Edge のクロスブラウザ。ヘッドレスで CI に載せやすく、テストスクリプトの自動生成も得意。Playwright 本体の長いエコシステムとドキュメントを引き継ぐ
- **弱い軸**：既定はクリーン環境なので、実ログイン状態を使うには storageState の事前取得など別途セットアップがいる。Lighthouse などの深い診断は持たない
- **有用なシーン**：複数ブラウザでの E2E テストを、CI で何度でも同じ結果で回したい
- 導入：[GitHub](https://github.com/microsoft/playwright-mcp)

## 数字とセキュリティの留保

公平のために、鵜呑みにしないほうがいい点も書いておきます。

**性能や優位の主張は、提供元由来のものが多い**点に注意がいります。たとえば agent-browser の「37% トークン削減」「95% 成功率」は Vercel 内部または追随メディアのベンチで、独立した第三者の再現は確認できませんでした（本番採用事例も今のところ見つかっていません）。同じく、どのツールの「得意」「最高」といった評価も、最後は自分の用途で試して確かめるのが確実です。

**セキュリティはどのツールも詰めるべき点があります**。Claude in Chrome は実ログイン状態を共有するぶん、操作権限の範囲や prompt injection の管理が要点になります。agent-browser にはバイナリ配布のチェックサム検証がない未修正 Issue（#1422）、chrome-devtools-mcp には `/tmp` の symlink following 脆弱性（CVE-2026-53765）が報告されています。Playwright MCP も CI 上でのシークレットの扱いに注意がいる。いずれも、組織導入では権限とシークレットの管理を前提に評価してください。

## まとめ

それぞれのツールに向き不向きがあります。特徴を把握して、用途に合ったものを選んでください。
