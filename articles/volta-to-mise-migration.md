---
title: "Node.js バージョン管理、Volta から mise へ移行した。Python は uv 管理のままにした。"
emoji: "🐸"
type: "tech"
topics: ["nodejs", "mise", "volta", "uv", "claudecode"]
published: true
---

Volta から mise への移行メモです。

Node.js のバージョン管理には数年 Volta を使ってきました。ただ、リリースが v2.0.2（2024 年 12 月）で止まっているのがずっと気になっていて、他のツールへ移りたい気持ちはありつつ、動いている環境を壊してまで、とは思えず先延ばしにしていました。

重い腰を上げたきっかけは、GitHub の README に追加されていたこの一文です。

> Volta is unmaintained.

今動くものは当面動くけれど、OS やエコシステムの変化による破損には対応できない、という趣旨の注意書きです。さらにピン留めされた Issue #2080 では、メンテナ自身が「移行先として mise を推奨する。メンテナの大半が今は mise を使っている」と表明していました。
https://github.com/volta-cli/volta/issues/2080

環境は macOS (Apple Silicon) + Homebrew、mise 2026.7.12。情報は 2026 年 7 月時点の調査に基づきます。対象読者は「Volta を使っていて、この先どうするか考え始めた人」です。

## 移行先に mise を選んだ理由

Node のバージョン管理だけなら nvm/fnm/asdf など候補は他にもあります。
以下 4 つの理由で mise を選びました。

1. **利用者が多い** ─ 情報が見つかりやすく、ハマっても先人がいる
2. **更新が続いている** ─ 2026 年 7 月だけでも複数のリリース
3. **Volta メンテナが名指しで推奨する移行先** ─ 公式のお墨付き
4. **様々なツールに対応** ─ Node 以外にも[大量のツール](https://mise.jdx.dev/registry.html#tools)やグローバル CLI に対応

## 移行のメモ

作業は Claude Code に委任し、私がやったのは方針の判断と承認だけでした。

まず全プロジェクトの棚卸しから。`volta` キーを持つ package.json を grep。「あちこちでピンしているから大変」というのは思ってましたが、実際は数えるほどしかありませんでした。移行を迷っている人は、まず棚卸しからをおすすめします。

注意点としては、mise は package.json の `volta` キーを読みません。なので各プロジェクトに `.node-version` を置いて変換します。また、mise はこのファイルを既定では読まないので、明示的な有効化が必要です。

手順のダイジェストはこれだけです (バージョンなどは環境に合わせる)。

```bash
# インストールとシェル連携（PATH 操作の最後に置く）
brew install mise
echo 'eval "$(mise activate zsh)"' >> ~/.zshrc

# .nvmrc / .node-version を読む設定（既定は無効）
mise settings add idiomatic_version_file_enable_tools node

# Node のインストールとデフォルト設定
mise install node@22.23.1 node@24.15.0 node@25.1.0 node@26.4.0
mise use -g node@26.4.0

# グローバル CLI の再構築
mise use -g pnpm@10.20.0 npm:wrangler@4.27.0 npm:typescript@5.9.2 # ほか多数
```

Volta 管理だったグローバル CLI 22 個も mise 側へ再構築しました（pnpm は corepack をやめて mise 直接管理に切替）。ここで 1 つつまずきが。mise の npm バックエンドは、週間ダウンロード数が 1,000 未満のパッケージをサプライチェーン対策として拒否します。手元では週 18 DL と週 617 DL のニッチな CLI が弾かれました。`MISE_NPM_SHELL_OUT=1` を付けて素の npm に処理を切り替えたら通りました。

最後に、各プロジェクトで `node --version` を実測してピンどおりに解決されることを確認し、`~/.volta` を削除。容量は 6.1GB で、ちょっとした大掃除になりました。

## Python は uv のまま

mise は Python なども管理できます。では Python のバージョン管理も mise に寄せるべきか。私はそうしませんでした。

uv は Python 本体のインストール・venv・パッケージ・lockfile を単独で完結できます。`uv run` すれば必要な Python を勝手に取ってきますし、`.python-version` の読み書きも自前でやる。ここに mise を挟んでもメリットがありません。

ただし、Node と Python のピンを `mise.toml` 1 枚に集約したい polyglot なリポジトリを扱うときと、uv を使わない Python ツールチェーンが必要になったときは採用したら良いと思います。

結果、役割分担はこう整理しました。

- **mise:** Node ランタイム + グローバル CLI
- **uv :** Python のすべて（本体・venv・パッケージ）
- **pnpm:** Node プロジェクトのパッケージ

## まとめ

- Volta は unmaintained。使えなくなる前に移行しておくのがお勧めです
- mise は volta キーを読みません。`.node-version` への変換と idiomatic 設定の有効化を忘れずに
- npm バックエンドに拒否されたら `MISE_NPM_SHELL_OUT=1`
- Python で uv を使ってる場合は本文中の条件でない限りそのまま

今は AI が面倒な作業を肩代わりしてくれるので、こういった移行も楽ですね。
