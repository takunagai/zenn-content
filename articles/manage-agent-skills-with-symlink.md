---
title: "複数の AIコーディングツールのスキルを symlink で一元管理する"
emoji: "🐸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [claudecode, codex, opencode, ai, 開発環境]
published: true
---

Claude Code、Codex、Gemini CLI、Cursor、GitHub Copilot、Windsurf … Agent Skills (以下、スキル) に対応した AIコーディングツールは40以上。複数を併用している方も多いのではないでしょうか。

選択肢があったり使い分けができることは良いのですが、ツールごとにスキルの配置先が異なるため、共通のスキルを全ツールで使おうとすると管理が一気に面倒になります。コピーして手動同期する運営だと、３日で破綻するでしょう。

[`npx skills`](https://skills.sh/) のようなパッケージマネージャを使えばリポジトリからのインストールと配信は自動化されます。しかし、**自作スキルの管理まではやってくれません**。自作スキルも含めて全ツールに確実に配信するには、エコシステムの仕組みを理解した上で補助スクリプトで整えるのが効率的です。

この記事では、Agent Skills エコシステムが採用する **`~/.agents/skills/` を SSOT（Single Source of Truth）にした symlink (シンボリックリンク) 配信の仕組み**と、その実践的なセットアップを紹介します。

## 最終構成

スキルの実体は `~/.agents/skills/` の１箇所だけで、大半のツールはディレクトリ symlink、Claude Code だけは `.system/` が同居するため個別 symlink です。

```
~/.agents/skills/              ← 実体: SSOT + Git 管理
│
├─ ~/.claude/skills/           ← 個別 symlink + .system/
├─ ~/.codex/skills             → ../.agents/skills（ディレクトリ symlink）
├─ ~/.config/opencode/skills   → ../../.agents/skills
├─ ~/.windsurf/skills          → ../.agents/skills
├─ ~/.cursor/skills            → ../.agents/skills
├─ ~/.copilot/skills           → ../.agents/skills
├─ ~/.gemini/skills            → ../.agents/skills
└─ ~/.gemini/antigravity/skills   → ../../.agents/skills
```

[Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills) のファイル形式（`SKILL.md`）は共通ですが、配置場所はツールごとにバラバラです。エコシステムは `~/.agents/skills/` を SSOT とし symlink で参照する規約に収束しており、パッケージマネージャ [`npx skills`](https://skills.sh/)（[vercel-labs/skills](https://github.com/vercel-labs/skills)）もこの規約に従っています。

## npx skills

### `npx skills add`によるスキルのインストールフロー

ターミナルで `npx skills add` を実行すると、対話形式でインストールが進みます。
途中、エージェント選択がありますが、この記事の方法ではディレクトリ symlink 経由で SSOT を直接参照しているので、**「Claude Code」だけ選びます。** インストール方式は、「Symlink (Recommended)」を選択。グローバルにインストールするかは任意で。
最後に Gen, Socket, Snyk によるセキュリティリスク評価も出るので確認しておきましょう。

### `npx skills` と自作スキルの共存

`npx skills` は `~/.agents/.skill-lock.json` で自分がインストールしたスキルを追跡しています。自作スキルや手動で追加したスキルはこのロックファイルに登録されないため、`npx skills update` を実行しても**自作スキルには一切触れません**。安心して共存できます。

:::message alert
⚠️注意：**`npx skills add` で自作スキルと同名のスキルをインストールすると、警告なしで上書きされます。** 衝突検出の仕組みはないので、自作スキル名には接頭辞や接尾辞を付けるなど公開スキルと被りにくい命名を推奨します。
:::

## symlink の方式: ディレクトリ symlink vs 個別 symlink

`npx skills` の公式実装では、全エージェントに対して**スキルごとに１本ずつ個別 symlink** を作成します。ディレクトリ symlink は使いません。

一方、この記事では大半のツールに**ディレクトリ symlink**（`~/.codex/skills → ../.agents/skills`）を使っています。これは公式とは異なりますが、メンテナンスが楽で実用上は問題ありません。スキルを追加しても symlink を張り直す必要がないからです。`npx skills add -g` でインストールしたスキルも、SSOT に配置された時点でディレクトリ symlink 経由で各ツールから見えるため、共存できます。

**ただし Claude Code だけはディレクトリ symlink が使えません。**

Claude Code の `~/.claude/skills/` には、ユーザースキル以外のものが同居しているからです。

```
~/.claude/skills/
  ├── .system/            ← スキル管理ツール（skill-creator, skill-installer）
  ├── '_plugins のスキル'  ← プラグインメタデータ
  ├── git-workflow/        → ../../.agents/skills/git-workflow（symlink）
  └── quality-check/       → ../../.agents/skills/quality-check（symlink）
```

`.system/` にはスキルの作成・インストールを支援するシステムスキルが入っています。ディレクトリ丸ごと symlink にすると `.system/` の置き場がなくなりますし、逆に `.system/` を SSOT に入れると他のツールが予期しないファイルを読んでしまいます。

**SSOT は共有スキルだけの純粋な場所に保つ。** この原則に従って、Claude Code だけは `~/.claude/skills/` を実ディレクトリのまま残し、中に個別 symlink を並べます。

:::message
`impeccable:polish` や `document-skills:pdf` のようなコロン付きスキルは Claude Code 独自のプラグインシステムなので、今回の一元管理の対象外です。
:::

## セットアップ

すでに `~/.claude/skills/` に実ディレクトリとしてスキルがある状態から、ここで解説している SSOT に移行する手順です。

⚠️ ご使用は自己責任で。バックアップは必ず取ってください。

### Step 1: バックアップ

```bash
cp -a ~/.claude/skills/ ~/.claude/skills.backup.$(date +%Y%m%d)
```

### Step 2: スキルを SSOT に移動

Finder で `~/.claude/skills/` を開き、`.system/` と既存の symlink を除くスキルフォルダを `~/.agents/skills/` へ移動します。

### Step 3: symlink を作成

ホームディレクトリの場所に依存しないよう、相対パスを使います。

```bash
# Claude Code: 個別 symlink
for skill in ~/.agents/skills/*/; do
  name=$(basename "$skill")
  [ "${name:0:1}" = "." ] && continue
  [ -L ~/.claude/skills/"$name" ] && continue
  ln -s "../../.agents/skills/$name" ~/.claude/skills/"$name"
done

# 他のツール: ディレクトリ symlink（既存がなければ作成）
[ -L ~/.codex/skills ] || ln -s ../.agents/skills ~/.codex/skills
[ -L ~/.config/opencode/skills ] || ln -s ../../.agents/skills ~/.config/opencode/skills
[ -L ~/.gemini/skills ] || ln -s ../.agents/skills ~/.gemini/skills
[ -L ~/.cursor/skills ] || ln -s ../.agents/skills ~/.cursor/skills
[ -L ~/.copilot/skills ] || ln -s ../.agents/skills ~/.copilot/skills
[ -L ~/.windsurf/skills ] || ln -s ../.agents/skills ~/.windsurf/skills
[ -L ~/.gemini/antigravity/skills ] || ln -s ../../.agents/skills ~/.gemini/antigravity/skills
```

### Step 4: 検証

```bash
for tool in claude codex gemini cursor copilot windsurf; do
  ls ~/."$tool"/skills/git-workflow/SKILL.md 2>/dev/null \
    && echo "$tool: OK" \
    || echo "$tool: FAIL"
done

# 別パスのツール
ls ~/.config/opencode/skills/git-workflow/SKILL.md 2>/dev/null \
  && echo "opencode: OK" \
  || echo "opencode: FAIL"

ls ~/.gemini/antigravity/skills/git-workflow/SKILL.md 2>/dev/null \
  && echo "antigravity: OK" \
  || echo "antigravity: FAIL"
```

全部 `OK` なら完了です。

## 運用

`~/.agents/skills/` にスキルを追加・削除したとき、ディレクトリ symlink のツール（Codex、Gemini CLI、Cursor など）は何もする必要がありません。Claude Code だけは個別 symlink の操作が必要です。

```bash
# 追加
ln -s "../../.agents/skills/my-new-skill" ~/.claude/skills/my-new-skill

# 削除
rm ~/.claude/skills/my-old-skill
```

## 管理スクリプトで「ズレない仕組み」を作る

上記の手動操作を自動化するスクリプトです。全ツールの symlink 状態を１コマンドで検証・修復できます（⚠️ 自己責任で）。

:::details setup-skills.sh（クリックで展開）

```bash:~/.agents/setup-skills.sh
#!/usr/bin/env bash
#
# setup-skills.sh — SSOT (~/.agents/skills/) から各ツールへ symlink を配信・検証
#
# 使い方:
#   bash ~/.agents/setup-skills.sh          # symlink を作成・修復
#   bash ~/.agents/setup-skills.sh --check  # 検証のみ（変更なし）
#
set -euo pipefail

SSOT="$HOME/.agents/skills"
CHECK_ONLY=false
[[ "${1:-}" == "--check" ]] && CHECK_ONLY=true

if [[ ! -d "$SSOT" ]]; then
  echo "ERROR: SSOT not found: $SSOT" >&2
  exit 1
fi

green()  { printf '\033[32m  ✓ %s\033[0m\n' "$1"; }
yellow() { printf '\033[33m  ! %s\033[0m\n' "$1"; }
red()    { printf '\033[31m  ✗ %s\033[0m\n' "$1"; }

errors=0

# ── Claude Code: 個別 symlink ──
setup_claude() {
  local dir="$HOME/.claude/skills"
  echo "── Claude Code（個別 symlink）──"

  if [[ ! -d "$dir" ]]; then
    yellow "SKIP: $dir not found"
    return
  fi

  for skill in "$SSOT"/*/; do
    local name
    name=$(basename "$skill")
    [[ "${name:0:1}" == "." ]] && continue

    local link="$dir/$name"
    local expected="../../.agents/skills/$name"

    if [[ -L "$link" && -e "$link" ]]; then
      local actual
      actual=$(readlink "$link")
      if [[ "$actual" == "$expected" ]]; then
        green "OK: $name"
      else
        if $CHECK_ONLY; then
          yellow "MISMATCH: $name → $actual"
          errors=$((errors + 1))
        else
          rm "$link"
          ln -s "$expected" "$link"
          green "FIXED: $name"
        fi
      fi
    elif [[ -L "$link" ]]; then
      if $CHECK_ONLY; then
        red "BROKEN: $name → $(readlink "$link")"
        errors=$((errors + 1))
      else
        rm "$link"
        ln -s "$expected" "$link"
        green "FIXED: $name"
      fi
    elif [[ -d "$link" ]]; then
      red "CONFLICT: $name は実ディレクトリ（手動で SSOT に移動してください）"
      errors=$((errors + 1))
    else
      if $CHECK_ONLY; then
        yellow "MISSING: $name"
        errors=$((errors + 1))
      else
        ln -s "$expected" "$link"
        green "CREATED: $name"
      fi
    fi
  done
}

# ── 他ツール: ディレクトリ symlink ──
setup_dir_symlink() {
  local tool="$1" link="$2" expected="$3"
  local parent
  parent=$(dirname "$link")

  if [[ ! -d "$parent" ]]; then
    yellow "SKIP: ${tool}（${parent} なし）"
    return
  fi

  if [[ -L "$link" && -e "$link" ]]; then
    local actual
    actual=$(readlink "$link")
    if [[ "$actual" == "$expected" ]]; then
      green "OK: $tool → $actual"
    elif $CHECK_ONLY; then
      yellow "MISMATCH: $tool → $actual (expected $expected)"
      errors=$((errors + 1))
    else
      rm "$link"
      ln -s "$expected" "$link"
      green "FIXED: $tool"
    fi
  elif [[ -L "$link" ]]; then
    if $CHECK_ONLY; then
      red "BROKEN: $tool → $(readlink "$link")"
      errors=$((errors + 1))
    else
      rm "$link"
      ln -s "$expected" "$link"
      green "FIXED: $tool"
    fi
  elif [[ -d "$link" ]]; then
    red "REAL_DIR: $tool — $link は実ディレクトリ（SSOT に移動してください）"
    errors=$((errors + 1))
  elif $CHECK_ONLY; then
    yellow "MISSING: $tool"
    errors=$((errors + 1))
  else
    ln -s "$expected" "$link"
    green "CREATED: $tool"
  fi
}

# ── 実行 ──
setup_claude

echo ""
echo "── 他ツール（ディレクトリ symlink）──"
setup_dir_symlink "codex"       "$HOME/.codex/skills"            "../.agents/skills"
setup_dir_symlink "opencode"    "$HOME/.config/opencode/skills"  "../../.agents/skills"
setup_dir_symlink "gemini"      "$HOME/.gemini/skills"           "../.agents/skills"
setup_dir_symlink "cursor"      "$HOME/.cursor/skills"           "../.agents/skills"
setup_dir_symlink "copilot"     "$HOME/.copilot/skills"          "../.agents/skills"
setup_dir_symlink "windsurf"    "$HOME/.windsurf/skills"         "../.agents/skills"
setup_dir_symlink "antigravity" "$HOME/.gemini/antigravity/skills" "../../.agents/skills"

# ── サマリー ──
echo ""
skill_count=$(find "$SSOT" -mindepth 1 -maxdepth 1 -type d ! -name '.*' | wc -l | tr -d ' ')
echo "SSOT スキル数: $skill_count"
if [[ $errors -eq 0 ]]; then
  green "All good!"
else
  if $CHECK_ONLY; then
    red "要修正: $errors 件（--check なしで実行すると自動修復します）"
    exit 1
  else
    red "要対応: $errors 件（上記の CONFLICT / REAL_DIR を確認してください）"
  fi
fi
```

:::

使い方はシンプルです。

```bash
# 検証のみ（何も変更しない）
~/.agents/setup-skills.sh --check

# 検証 + 自動修復
~/.agents/setup-skills.sh
```

スキルの追加・削除後にこのスクリプトを１回実行すれば、Claude Code 用の個別 symlink が自動で同期されます。

インストールしていないツールは自動でスキップされるので、全部入れていなくても問題ありません。

## プロジェクト固有スキルにも使える

ホームレベルの SSOT はグローバルなスキル管理ですが、プロジェクト単位でも同じ構造が動きます。

```
my-saas-project/
├── .agents/skills/          ← プロジェクトローカルの SSOT
│   ├── copywriting/
│   └── seo-audit/
├── .claude/skills/          ← 個別 symlink → .agents/skills/
├── .windsurf/skills/        ← 個別 symlink → .agents/skills/
└── skills-lock.json         ← バージョン管理用ロックファイル
```

`npx skills add` をプロジェクト内で実行すると、この構造が自動生成されます。リポジトリにコミットすれば、チームメンバーが clone するだけで同じスキルセットが揃います。ホームレベルが「個人の道具箱」、プロジェクトレベルが「チームの共有道具箱」。同じアーキテクチャです。

## Git 管理

自分は `~/.agents/` を Git 管理するようにしました。今回の設定で `.system` ディレクトリ以外のスキルは symlink での管理としたため、以下を付け足しました。

```.gitignore
/skills/*
!/skills/.system/
```

## まとめ

本来は各ベンダーが `~/.agents/skills/` を直接読んでくれれば済む話で、symlink で橋渡しすること自体が余計な手間です。とはいえ現状そうなっていない以上、SSOT + symlink で一元管理するのが現実的な落としどころです。
