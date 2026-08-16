---
title: "mise チートシート ─ 開発環境の管理でよく使うコマンドまとめ"
emoji: "🐸"
type: "tech"
topics: ["mise", "nodejs", "pnpm", "cli", "devtools"]
published: true
---

![mise でランタイムのバージョンを棚のように管理するイラスト](/images/mise-command-cheatsheet/eyecatch.jpg)

mise を使うと、Node.js・pnpm・Python・Bun などのランタイムや開発CLI をまとめて管理できます。本記事では、mise を日常的に使ううえで覚えておきたい実用コマンドと基本的な運用方法をまとめます。

Volta からの乗り換えの経緯は、こちらの記事に書いています。

https://zenn.dev/takna/articles/volta-to-mise-migration

## ツールの役割分担

```text
Homebrew
└─ macOS系ツール・GUIアプリ・システム寄りCLI

mise
├─ Node.js
├─ pnpm
├─ Bun
├─ Python
├─ その他ランタイム
└─ グローバル開発CLI

pnpm
└─ 各Node.jsプロジェクトの依存パッケージ
```

### 基本方針
- ランタイムやグローバル開発CLI → mise
- プロジェクトのJS依存関係 → pnpm

なお筆者は、Python だけは mise に載せず uv 単独で管理しています。uv は Python 本体・仮想環境・パッケージまで一括で面倒を見てくれるため、mise 側に二重で持たせる利点が薄いからです。逆に、Node と Python のバージョンを `mise.toml` 一枚に集約したい polyglot なリポジトリでは、mise に寄せたほうが管理は楽になります。

## 基本的な考え方

- グローバル環境 → `mise use -g`
- プロジェクト単位 → `mise use`
- アップデート → `mise upgrade`
- 一時的に別バージョンを使う → `mise x`
- 状態確認 → `mise ls`
- 不要バージョン削除 → `mise prune`

## よく使うコマンドまとめ

- グローバルで最新版を導入：`mise use -g node@latest`
- プロジェクトでバージョン指定：`mise use node@22`
- 設定済み環境をインストール：`mise install`
- 1ツールを更新：`mise upgrade node`
- 全ツールを更新：`mise upgrade`
- メジャーバージョンも更新：`mise upgrade --bump node`
- 更新内容を事前に確認：`mise upgrade --dry-run`
- 一時的に別バージョンを使う：`mise x node@24 -- node -v`
- 現在の状態を確認：`mise ls`
- 現在有効なバージョンを確認：`mise current`
- 実行ファイルを確認：`mise which node`
- 最新バージョンを確認：`mise latest node`
- 更新可能か確認：`mise outdated`
- 利用可能バージョン一覧：`mise ls-remote node`
- 不要バージョン確認：`mise prune --dry-run`
- 不要バージョン削除：`mise prune`
- shim再生成：`mise reshim`
- miseの問題を診断：`mise doctor`

## おすすめ運用

### グローバル環境の場合

初期設定：

```bash
mise use -g node@latest
mise use -g pnpm@latest
mise use -g bun@latest
mise use -g python@latest
```

日常的なメンテナンス：

```bash
mise outdated
mise upgrade
mise prune --dry-run
mise prune
```

### プロジェクトの場合

バージョン指定：

```bash
mise use node@22
mise use pnpm@10
```

環境を再構築：

```bash
mise install
pnpm install
```

---

## 1. グローバルで最新版を使う
`-g` を付けると、PC全体のデフォルトとして設定される。

- Node.js：`mise use -g node@latest`
- pnpm：`mise use -g pnpm@latest`
- Bun：`mise use -g bun@latest`
- Python：`mise use -g python@latest`

特定バージョンなら：

- Node.js：`mise use -g node@24`
- pnpm：`mise use -g pnpm@10`

## 2. プロジェクトごとにバージョンを指定

- Node.js：`mise use node@22`
- pnpm：`mise use pnpm@10`

→ `mise.toml` に設定され、そのプロジェクトでは指定したバージョンが自動的に使われる。

```toml
[tools]
node = "22"
pnpm = "10"
```

グローバル → `mise use -g`
プロジェクト → `mise use`

## 3. ツールをアップデート

- pnpm：`mise upgrade pnpm`
- Node.js：`mise upgrade node`
- Bun：`mise upgrade bun`
- Python：`mise upgrade python`
- 設定済みツールをまとめて更新：`mise upgrade`

mise 管理下では、例えば `pnpm self-update` ではなく `mise upgrade pnpm` を使う。

更新前に内容を確認したいときは `--dry-run`（`-n`）を付ける。

```bash
mise upgrade --dry-run
```

## 4. メジャーバージョンも更新
設定が `node = "22"` なら、`mise upgrade node` は基本的に最新の22.xへ更新する。
メジャーバージョンを含めて設定自体を更新：`mise upgrade --bump node`

`--bump` は最新版をインストールしたうえで、`mise.toml` の記述もそのバージョンに書き換えてくれる。

## 5. mise.toml の環境をインストール
設定済みツールをまとめてインストール：`mise install`

例えば、

```toml
[tools]
node = "22"
pnpm = "10"
python = "3.13"
```

なら、必要なバージョンをまとめてインストールしてくれる。
以下、典型的な流れ。

```bash
git clone ...
cd project
mise install
pnpm install
```

## 6. 一時的に別バージョンを使う

- Node.js 24：`mise x node@24 -- node -v`
- Node.js 最新版：`mise x node@latest -- node -v`
- Python 3.14：`mise x python@3.14 -- python`

現在の設定を変更せず、そのコマンドだけ指定バージョンで実行できる。
`x` は `mise exec` のエイリアス。同様に `mise up`（upgrade）、`mise dr`（doctor）も使える。

## 7. 現在の状態を確認

- インストール済み・使用中ツール：`mise ls`
- 現在有効なバージョン：`mise current`
- Node.jsの実体：`mise which node`

トラブル時は `mise which node` と `which node` を比較すると、mise管理のNode.jsが本当に使われているか確認できる。

## 8. 最新・利用可能バージョンを確認

- Node.js最新版：`mise latest node`
- pnpm最新版：`mise latest pnpm`
- Python最新版：`mise latest python`
- Node.jsの利用可能バージョン一覧：`mise ls-remote node`

## 9. アップデート可能なツールを確認

更新可能なツールを確認：`mise outdated`
確認後、まとめて更新：`mise upgrade`

なお `mise outdated` は、既定では設定に書いたバージョン範囲の中だけを比較する。`node = "22"` なら 22.x の新版しか出てこない。メジャーバージョンを跨いだ更新候補まで見たい場合は `--bump`（`-l`）を付ける。

```bash
mise outdated --bump
```

## 10. 不要な旧バージョンを削除
Node.jsなどの古いバージョンが増えてきたときに便利。

- 削除対象だけ確認：`mise prune --dry-run`
- 実際に削除：`mise prune`

補足として、`mise upgrade` は既定で更新後に旧バージョンを削除する（残したい場合は `--no-prune`）。それでも `mise prune` が要るのは、`mise use -g node@latest` のようにバージョンを差し替えたときや、`mise x` で一時利用したバージョンが residual として残るケース。削除対象は `mise ls --prunable` でも確認できる。

## 11. コマンドが認識されない場合

shimを再生成：`mise reshim`
特に `pnpm -g` などでCLIを追加した後、コマンドが見つからない場合に使える。

## 12. npm製CLI も mise で管理
グローバルCLI も mise側へ集約できる (例: Biome)

- npmでグローバル管理：`npm install -g @biomejs/biome`
- miseで管理：`mise use -g npm:@biomejs/biome@latest`

---

## あとがき

mise では、mise use でツールやバージョンを設定し、mise upgrade で更新するのが基本です。ランタイムやグローバルCLIを mise に集約することで、開発環境のインストール・更新・バージョン管理をシンプルにできます。
