---
title: "AIエージェントと相性がいいデザインツールはどれ？Figma/Penpot/Pencil/Onlook比較"
emoji: "🐸"
type: "tech"
topics: [figma, penpot, pencil, onlook, ai駆動開発]
published: true
---

AI エージェントに UI を作らせて、最終的にコードへ落としたい。
あるいは、すでにあるデザイン資産を AI で整理・同期しながら運用したい。

そう考え始めると、Figma だけで十分なのか、Penpot や Pencil、Onlook も触っておくべきか、判断に迷います。結論を先に書きます。

- **デザインとコードを速く往復させたい** → Pencil / Onlook
- **既存の資産を保ちつつ AI で整理・運用したい** → Penpot
- **チーム標準と外部協業を最優先したい** → Figma に Figma Make を足す

「AI がデザインを読むだけでなく、書き換える」「コードベースをソース・オブ・トゥルースにする」という軸で見ると、Pencil と Onlook は Figma / Penpot より一歩踏み込んだ思想を持っています。この記事では、あなたが選ぶときに見る 3 つの軸——立ち位置・AI 連携の深さ・データの置き場——を、比較表とともに整理します。

## 4 つのツールの立ち位置

[Figma](https://www.figma.com/) は、クラウド型コラボの定番です。デザインからコードを生成する公式機能として [Figma Make](https://www.figma.com/make/) を持ち、フレームから HTML / CSS / React などの編集可能なコードを出せます。[Dev Mode](https://www.figma.com/dev-mode/) と組み合わせれば、開発側に仕様を渡す動線も整います。

[Penpot](https://penpot.app/) はオープンソースのデザインツールで、Docker などで [自社サーバーにセルフホスト](https://penpot.app/self-host) できるのが大きな特徴です。[Penpot MCP サーバー](https://penpot.app/penpot-mcp-server) を立てると、AI エージェントから design file の要素（content・library・comments 等）を読み書きできます。MCP サーバーそのものも自社環境に置けるため、外部クラウドに依存しない運用ができます。

[Pencil](https://www.pencil.dev/) は「コードベースで動く AI ネイティブデザインツール」です。`.pen` ファイルをリポジトリに置いて Git でバージョン管理・ブランチ・マージできます。MCP では読み取りだけでなく **書き込み権限（write access）** も提供されており、Claude Code などのエージェントからキャンバスを直接操作できます。提供形態はスタンドアローンのデスクトップアプリ（macOS / Windows / Linux）と、VSCode / Cursor / Windsurf 向けの IDE 拡張機能の両方で、料金は[現時点で無料](https://www.pencil.dev/pricing)です。

[Onlook](https://onlook.com/) は「Cursor for Designers」を掲げ、既存の React コードベースを取り込み、**実コードを source of truth にしたまま視覚的に編集する**思想のツールです。Next.js + Tailwind と相性が良く、セルフホスト版は [GitHub](https://github.com/onlook-dev/onlook) から無料で利用できます。クラウド版は Waitlist／問い合わせベースで提供されています。

## AI との往復をどう設計しているか

4つのツールは、AI にデザインをどう触らせるかという思想がはっきり違います。ざっくり並べると **Pencil ≒ Onlook ＞ Penpot ＞ Figma** という順です。

「往復の深さ」は、デザインからコード・コードからデザインの両方向で何ができるかで決まります。順に見ていきましょう。

### デザインからコードへ

Figma は **Figma Make** で、フレームから編集可能なコードを生成します。HTML / CSS / JavaScript / React などを出力でき、生成結果を同じスペースで微調整できるのが強みです。あくまで「生成支援」の枠で、コードベース側を直接編集する設計ではありません。

Penpot は **MCP 経由で HTML / CSS の生成やデザイントークンの抽出**ができます。[`penpot-export`](https://github.com/penpot/penpot-export) を使えば CSS / SCSS / JSON 形式でのトークン出力も可能で、デザインシステムの整合性を保ったままコード化したいケースに向きます。

Pencil は `.pen` ファイルをコードベースに置く設計なので、変換というより **Claude Code 経由で実装まで一気通貫**にしやすい構造です。デザインとコードが同じ Git の世界に同居します。

Onlook はそもそも変換が要りません。**最初から実コードを編集している**ため、デザインとコードのズレが構造的に発生しないのが最大の特徴です。

### コードからデザインへ

逆方向、つまりコードを起点としたデザイン変更では、Onlook と Pencil が突出します。

Onlook は既存の React コードベースをそのまま取り込み、コードを source of truth にしたままビジュアル編集します。コード側の変更がそのままデザインに反映されます。

Pencil は「コードベースに design files を置く」設計と、MCP の write access によって、**AI がコードとデザインの両側をまたいで調整**できます。エージェントがコードを書き換える流れでデザインも整える、という運用が想定されています。

Penpot はコードからの直接編集よりも、デザイン構造の抽出・反映・整合性維持が中心です。コードからデザインへの往復は補助的にとどまります。

Figma も Dev Mode や Make でコード側に寄る導線はありますが、コードベースをそのまま編集するワークフローとは別物です。

:::message
**「AI 連携が深い ＝ 良い」とは限らない**
Pencil / Onlook は AI と密結合する分、チーム全体での標準性や外部協業のしやすさで Figma に劣ります。あなたのチームが何を最優先したいかで、評価軸そのものが変わります。
:::

## 四者比較表

| 観点 | Figma | Penpot | Pencil | Onlook |
|---|---|---|---|---|
| 位置づけ | 業界標準のクラウドデザインツール | OSS デザインツール + MCP | コードベース連動の AI ネイティブデザインツール | 実コードを直接編集するデザインツール |
| AI との関係 | Figma Make でデザイン→コード支援 | MCP で読み書き両対応 | MCP で強い読み書き連携（write access） | コードを軸に AI と視覚編集 |
| デザイン→コード | HTML / CSS / React 等を生成 | HTML / CSS 生成・トークン抽出 | コードベースに同居、実装まで近い | そもそも実コード上で編集 |
| コード→デザイン | 限定的 | 補助的 | 双方向性が高い | 最も自然 |
| データの置き場 | Figma クラウド | クラウド／セルフホスト | Git リポジトリ（`.pen`） | 実コードのリポジトリ |
| セルフホスト | 不可（SaaS のみ） | 可（公式に Docker 等で推奨） | ローカル動作（デスクトップ／IDE 拡張） | 可（GitHub から無料） |
| 提供状況 | 正式提供 | 正式提供 | 現時点は無料公開 | クラウドは Waitlist、セルフホストは無料 |
| 向いている人 | 大規模コラボ、標準化重視 | OSS・自己ホスト・AI 運用重視 | エンジニア主導の設計／実装 | 開発とデザインを一体化したい人 |

注：Pencil はデスクトップアプリ／IDE 拡張、Onlook はコードベース上で動く設計のため、Figma / Penpot のような SaaS と「セルフホスト可否」を同じ軸で比べるのは概念がややズレます。便宜的な表記として読んでください。

## どれをどう選ぶか

**Figma** は、すでに Figma 中心で回っているチーム、デザイン組織が大きい、外部協業が多い場合にいちばん無難です。Figma Make を必要箇所だけ足せば、AI 連携も最低限カバーできます。

**Penpot** は、オープンソース・セルフホスト・デザインシステム管理・AI によるファイル保守を重視する場合に有力です。MCP サーバーも自社環境に置けるため、機密性の高いプロジェクトとも噛み合います。

**Pencil** は、Claude Code などのエージェントと組み合わせて、設計と実装を物理的に近づけたいエンジニア寄りのチームに刺さります。`.pen` ファイルが Git にコミットされるので、**デザイン変更を Pull Request のレビュー対象に乗せられる**のが、個人的にはいちばん面白い点です。

**Onlook** は、既存の React アプリを壊さずに、デザイナーがコードの中で直接作業したい場合に独特の価値があります。Next.js + Tailwind のプロジェクトなら、本番コードとデザインのズレが構造的に発生しないという利点が大きい。

筆者の率直な感触として、AI エージェントと深く組み合わせるなら **Pencil または Onlook を試す価値が高い**と感じています。Figma が劣っているわけではなく、両者は向き合っている課題そのものが違います。Figma / Penpot は「みんなで使えるデザイン基盤」、Pencil / Onlook は「実装を加速する個人〜小規模チーム向けの道具」と捉えると、選び分けやすくなります。

## まとめ

実務では、**Figma または Penpot を設計基盤として残しつつ、Pencil または Onlook を実装寄りの加速器として併用する**構成が、現時点ではいちばんバランスが良い選択です。

1 つのツールで完結させたいなら、あなたのチームが「標準性」と「往復速度」のどちらを取るかを先に決めましょう。AI 連携を最大化したいなら、コードベースに design file が同居する Pencil か、実コードを直接編集する Onlook を、まず触ってみてください。既存の Figma 資産を持っているなら、Penpot の MCP を立てて「AI に自社のデザイン資産を読ませる」ところから始めるのが現実的です。

「AI と一緒にデザインする」という言葉の中身は、ツールによってまったく違います。あなたの業務がどちら寄りなのかを見定めると、最初に触るべきツールが自ずと決まります。

## 参考

### 公式情報（一次情報）

- [Figma Make](https://www.figma.com/make/)
- [Figma Dev Mode](https://www.figma.com/dev-mode/)
- [Penpot](https://penpot.app/)
- [Penpot MCP サーバー](https://penpot.app/penpot-mcp-server)
- [Penpot セルフホスト](https://penpot.app/self-host)
- [penpot-mcp（GitHub）](https://github.com/penpot/penpot-mcp)
- [Pencil](https://www.pencil.dev/)
- [Pencil 料金](https://www.pencil.dev/pricing)
- [Pencil ダウンロード](https://www.pencil.dev/downloads)
- [Onlook](https://onlook.com/)
- [Onlook 料金](https://onlook.com/pricing)
- [onlook-dev/onlook（GitHub）](https://github.com/onlook-dev/onlook)

### 解説・比較記事

- [Pencil.dev 入門：Figma との違いと MCP 連携による Design-to-Code](https://zenn.dev/zenchaine/articles/pencil-io-vs-figma-design-tools-2026)
- [AI フロントエンドツール 5 選徹底比較 Lovable / v0 / Stitch / Figma...](https://zenn.dev/tenormusica/articles/ai-app-builder-comparison-2026)
- [新デザインツール Pencil はなぜエンジニアに刺さるのか](https://zenn.dev/aria3/articles/pencil-dev-why-engineers-love-it)
- [Figma からのコピペ。Pencil.dev でデザインをコードとして管理する](https://zenn.dev/yuukikawabata/articles/pencil-dev-introduction)
- [AI 時代に問い直すべきデザインツール選択基準（Yasuhisa Hasegawa）](https://yasuhisa.com/could/article/choosing-right-tools/)
- [定番デザインツールの Figma と注目の AI ネイティブな Pencil](https://sterfield.co.jp/blog/figma-to-pencil-with-claude-code/)
