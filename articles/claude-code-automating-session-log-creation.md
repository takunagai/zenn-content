---
title: "【Claude Code】AIとのやり取りをフックで自動記録。ナレッジとして活用しやすくする"
emoji: "🐸"
type: "tech"
topics: [claudecode, ai駆動開発]
published: true
---

★★TODO: Compactionだけでなく、手動 (`log#️⃣`) も必要
UserPromptSubmit　で前ステップのClaudeの応答の要約(あれば)とユーザープロンプト原文を保存するのが良いか？

:::message
**この記事でできるようになること**
- **自分のプロンプト**と**AIの応答の要約**を自動でログファイルに保存
- Compaction が発生しても**自分の指示を原文で保存**する
- 整形されたセッション履歴を**検索可能なナレッジ**として蓄積
:::

「あの時どんなプロンプトを書いたっけ？」
Claude Code で過去にうまくいったプロンプトを再現したいけど完全に思い出せずヤキモキする。そんな経験ありませんか？

AIとの対話履歴は、整形して蓄積しておくことで後で参照できるナレッジになります。以下のログサンプルように、「自分の指示原文」と「AIの応答の要約」をステップごとに保存することで、すっきり見やすいログを自動で出力する方法を解説します。

`````markdown
# セッションログ: Astro Skill 6.0アップグレード

**日付**: 2026年01月18日

## Step 1: 対応バージョンの確認

### 私の指示
```
このプロジェクトが提供する Astro Code Review Skill、対応している Astro のバージョンは？
```

### Claude の応答（要約）
- SKILL.md を分析し、明示的なバージョン指定はないが機能から Astro 5.x を暗黙的にターゲットと判断
- `render()` 関数形式、`server:defer`、`getLiveEntry()` など 5.x 機能を使用
- Astro 6.0 の breaking changes には未対応の状態

### 決定事項
- 現状は Astro 5.x 対応（暗黙的）

---

## Step 2: Astro 6.0+ アップグレード計画依頼

### 私の指示
```
現在ベータ版である Astro 6.0+ 対応としたい。そのための参考ドキュメントは以下。慎重に丁寧に計画を立てて。ultrathink
https://astro.build/blog/astro-6-beta/
https://v6.docs.astro.build/en/guides/upgrade-to/v6/
https://github.com/withastro/astro/tree/next
```

### Claude の応答（要約）
- Astro 6.0 Beta ブログと Upgrade Guide をスクレイピングして分析
- 主要な Breaking Changes を特定（Node 22+、Vite 7、Zod 4、削除API多数）
- Plan Mode に入り、詳細な実装計画を `/Users/takna/.claude/plans/reflective-doodling-haven.md` に作成

### 決定事項
- Astro 6.0+ 専用スキルとしてアップグレードする方針

---

## Step 3: Astro 6.0 メイン対応の確認

<同様のフォーマットで続く>
`````

## 実装手順

### 前提条件

- Claude Code がインストール済み
- `jq` コマンドがインストール済み

`jq` が未インストールの場合は、以下でインストールしてください：

```bash
# macOS (Homebrew)
brew install jq

# Ubuntu/Debian
sudo apt-get install jq

# その他の環境
# https://jqlang.github.io/jq/download/ を参照
```

### Step 1: スクリプトの作成

`~/.claude/hooks/pre-compact-log.sh` を作成します。

```bash
#!/bin/bash
# PreCompactフック: Compaction前にセッション会話履歴を自動保存
#
# 入力JSON構造:
#   {
#     "transcript_path": "/path/to/session.jsonl",
#     "session_id": "uuid",
#     "trigger": "auto" | "manual"
#   }
#
# 出力: Log/Session/YYYYMMDD_HHMM_session.md

set -euo pipefail

# JSON入力を読み取り
read -r INPUT_JSON

# jqが必要
if ! command -v jq &> /dev/null; then
    echo '{"status": "error", "message": "jq is required"}' >&2
    exit 0
fi

# 入力からフィールドを抽出
TRANSCRIPT_PATH=$(echo "$INPUT_JSON" | jq -r '.transcript_path // empty')
SESSION_ID=$(echo "$INPUT_JSON" | jq -r '.session_id // "unknown"')
TRIGGER=$(echo "$INPUT_JSON" | jq -r '.trigger // "unknown"')

# transcript_pathが空または存在しない場合は終了
if [ -z "$TRANSCRIPT_PATH" ] || [ ! -f "$TRANSCRIPT_PATH" ]; then
    echo '{"status": "skip", "message": "transcript_path not found"}'
    exit 0
fi

# プロジェクトパスを transcript_path から推測
# 例: /Users/takna/.claude/projects/-Users-takna-Projects-example/session.jsonl
PROJECT_PATH=$(dirname "$TRANSCRIPT_PATH" | sed 's|.*/projects/||' | sed 's/-/\//g')
if [ "$PROJECT_PATH" = "$(dirname "$TRANSCRIPT_PATH")" ]; then
    PROJECT_PATH="unknown"
fi

# ログディレクトリ（CLAUDE_PROJECT_DIRがあればそこ、なければ~/.claude）
BASE_DIR="${CLAUDE_PROJECT_DIR:-$HOME/.claude}"
LOG_DIR="${BASE_DIR}/Log/Session"
mkdir -p "$LOG_DIR"

# ファイル名: YYYYMMDD_HHMM_session.md
DATETIME=$(date '+%Y%m%d_%H%M')
DATE_FORMATTED=$(date '+%Y年%m月%d日 %H:%M')
LOG_FILE="${LOG_DIR}/${DATETIME}_session.md"

# セッションIDが既に存在するファイルがあれば、そのファイル名を使う（上書き防止）
EXISTING_FILE=$(ls "${LOG_DIR}"/*"${SESSION_ID:0:8}"* 2>/dev/null | head -1 || true)
if [ -n "$EXISTING_FILE" ]; then
    LOG_FILE="$EXISTING_FILE"
fi

# 一時ファイル
TEMP_FILE=$(mktemp)
trap "rm -f $TEMP_FILE" EXIT

# ヘッダーを書き込み
cat > "$TEMP_FILE" << EOF
# セッション会話履歴

**セッションID**: ${SESSION_ID}
**プロジェクト**: ${PROJECT_PATH}
**保存日時**: ${DATE_FORMATTED}
**トリガー**: ${TRIGGER}

---

EOF

# JJSONLファイルを解析してメッセージを抽出
# type: "user" または "assistant"、かつ isMeta: false のものを抽出
STEP=0
while IFS= read -r line; do
    # 空行はスキップ
    [ -z "$line" ] && continue

    # JSONとして解析
    MSG_TYPE=$(echo "$line" | jq -r '.type // empty' 2>/dev/null || true)
    IS_META=$(echo "$line" | jq -r '.isMeta // false' 2>/dev/null || true)
    TIMESTAMP=$(echo "$line" | jq -r '.timestamp // empty' 2>/dev/null || true)

    # isMeta: true はスキップ
    if [ "$IS_META" = "true" ]; then
        continue
    fi

    # ユーザーメッセージ
    if [ "$MSG_TYPE" = "user" ]; then
        STEP=$((STEP + 1))
        TIME_PART=$(echo "$TIMESTAMP" | sed 's/.*T\([0-9:]*\)\..*/\1/' 2>/dev/null || echo "")

        echo "" >> "$TEMP_FILE"
        echo "## Step ${STEP}${TIME_PART:+ (${TIME_PART})}" >> "$TEMP_FILE"
        echo "" >> "$TEMP_FILE"
        echo "### User" >> "$TEMP_FILE"
        echo "" >> "$TEMP_FILE"
        echo '`````' >> "$TEMP_FILE"

        # message.content を抽出（文字列または配列の場合がある）
        CONTENT=$(echo "$line" | jq -r '
            if .message.content | type == "string" then
                .message.content
            elif .message.content | type == "array" then
                [.message.content[] |
                    if type == "string" then .
                    elif .type == "text" then .text
                    else empty
                    end
                ] | join("\n")
            else
                ""
            end
        ' 2>/dev/null || echo "")

        echo "$CONTENT" >> "$TEMP_FILE"
        echo '`````' >> "$TEMP_FILE"
        echo "" >> "$TEMP_FILE"
    fi

    # アシスタントメッセージ
    if [ "$MSG_TYPE" = "assistant" ]; then
        echo "### Assistant" >> "$TEMP_FILE"
        echo "" >> "$TEMP_FILE"

        # message.content を抽出
        CONTENT=$(echo "$line" | jq -r '
            if .message.content | type == "string" then
                .message.content
            elif .message.content | type == "array" then
                [.message.content[] |
                    if type == "string" then .
                    elif .type == "text" then .text
                    else empty
                    end
                ] | join("\n")
            else
                ""
            end
        ' 2>/dev/null || echo "")

        # 長すぎる場合は省略しない（生データ保存のため）
        echo "$CONTENT" >> "$TEMP_FILE"
        echo "" >> "$TEMP_FILE"
        echo "---" >> "$TEMP_FILE"
    fi
done < "$TRANSCRIPT_PATH"

# Stepが0の場合（会話がない）はファイルを作成しない
if [ "$STEP" -eq 0 ]; then
    echo '{"status": "skip", "message": "no conversation found"}'
    exit 0
fi

# ファイルを保存
mv "$TEMP_FILE" "$LOG_FILE"

# 成功を返す
echo "{\"status\": \"ok\", \"file\": \"${LOG_FILE}\", \"steps\": ${STEP}}"
```

スクリプトに実行権限を付与します：

```bash
chmod +x ~/.claude/hooks/pre-compact-log.sh
```

### Step 2: settings.json の設定

`~/.claude/settings.json` に PreCompact フックを登録します。

既存の `settings.json` がない場合は新規作成、ある場合は `hooks` セクションに追加してください。

```json
{
  "hooks": {
    "PreCompact": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/pre-compact-log.sh",
            "timeout": 60000
          }
        ]
      }
    ]
  }
}
```

:::message
**設定項目の説明**
- `matcher`: 空文字列で全ての Compaction に対応
- `timeout`: タイムアウト時間（ミリ秒）。大きなセッションでも余裕を持って 60 秒に設定
:::

### Step 3: 動作確認

設定が完了したら、Claude Code で `/compact` コマンドを実行して手動で Compaction を発動させます。（※もちろん、Claude Code が自動で Compaction を行う場合も、同様にフックが発動してログが保存されます。）

```
/compact
```

成功すると、`~/.claude/Log/Session/` ディレクトリに Markdown ファイルが作成されます。

```bash
ls -la ~/.claude/Log/Session/
# 20260118_1030_session.md のようなファイルが作成されていればOK
```

- - -

## 解説

（以下、使う分には関係ないので、仕組みを知りたい方のみお読みください）

コンテキストの余裕がなくなり Compaction が発生すると、詳細な会話履歴は要約に置き換わり、**原文は失われます**。つまり、作業の継続には支障がないのですが、全ての原文をログに取っときたいときに困ります。

`UserPromptSubmit` フックを使えば、ユーザーがプロンプトを送信するときに実行できるため、ユーザーのプロンプト原文の保存はできます。しかし、自動 Compaction が発生するタイミングは Claude Code の作業中なので、ユーザーのプロンプト原文と AI の応答をセットで保存することができません。

なので、Compaction が発生する前に発動する `PreCompact` フックを使って、会話履歴を保存する仕組みを構築しました。

一度設定すれば、以後は完全自動。Compaction のたびに、あなたの指示原文と Claude の応答が Markdown ファイルとして保存され、いつでも検索・参照できるナレッジになります。

Claude Code を使った開発セッションの記録を自動化し、振り返りを効率化する方法を解説します。

**この記事では、PreCompact フックを使って会話履歴を自動保存する仕組みを構築します。**

一度設定すれば、以後は完全自動。Compaction のたびに、あなたの指示原文と Claude の応答が Markdown ファイルとして保存され、いつでも検索・参照できるナレッジになります。

## Compaction とは？

まず、Compaction について簡単に説明します。

Claude Code は、会話が長くなるとコンテキストウィンドウ（Claude が一度に処理できる情報量）の限界に近づきます。このとき、Claude Code は古い会話を**要約**して圧縮することで、新しいやり取りを続けられるようにします。これが「Compaction」です。

### Compaction が発生すると何が起きる？

1. **会話履歴が圧縮される**: 詳細なやり取りが要約文に置き換わる
2. **原文が失われる**: ユーザーの指示原文、Claude の応答原文が参照不能に
3. **コンテキストは維持**: 作業の継続には問題なし（Claude は要約を理解している）

つまり、**作業の継続には支障がない**のですが、**後で振り返りたいときに困る**という問題があります。

### 具体的にどう困る？

- 「どんなプロンプトで良い結果が出たか」を再現できない
- 技術的な議論の経緯を後から追えない
- チームメンバーへの共有時に「どんなやり取りだったか」を説明できない
- 自分の指示の改善点を分析できない

## 解決策: PreCompact フック

Claude Code には「フック（Hooks）」という機能があります。特定のイベントが発生したときに、任意のスクリプトを自動実行できる仕組みです。

今回使用する **PreCompact フック**は、Compaction が実行される**直前**に発火します。このタイミングで会話履歴を保存すれば、圧縮される前の完全な履歴を残せます。

### このソリューションの特徴

| 項目 | 内容 |
|------|------|
| 動作タイミング | Compaction 直前（自動） |
| 出力形式 | Markdown（人間が読みやすい） |
| ファイル構成 | 1セッション = 1ファイル |
| 保存内容 | ユーザー指示・Claude応答の原文 |
| 手間 | 初期設定のみ（以後は完全自動） |

## 出力されるログの形式

保存されるログファイルは以下のような形式です：

```markdown
# セッション会話履歴

**セッションID**: 302babbf-9ea4-4767-be2a-14feda8e115b
**プロジェクト**: /Users/takna/Projects/my-app
**保存日時**: 2026年01月18日 10:30
**トリガー**: auto

---

## Step 1 (09:15:32)

### User

`````

このコンポーネントにダークモード対応を追加して

`````

### Assistant

ダークモード対応を追加いたします。以下の変更を行います：
1. CSS カスタムプロパティでカラー定義
2. prefers-color-scheme メディアクエリの追加
...

---

## Step 2 (09:18:45)

### User

`````

テストも書いて

`````

### Assistant

ユニットテストを追加いたします。
...

---
```

### 各フィールドの説明

| フィールド | 説明 |
|-----------|------|
| セッションID | Claude Code が割り当てる一意の識別子 |
| プロジェクト | 作業していたディレクトリのパス |
| 保存日時 | ログが保存された日時 |
| トリガー | `auto`（自動）または `manual`（`/compact` コマンド） |
| Step | ユーザーの指示ごとに番号が振られる |
| User | ユーザーの入力（原文のまま） |
| Assistant | Claude の応答（原文のまま） |

## 活用シナリオ

### 1. ナレッジとしての活用

保存されたセッションログは、過去の作業を検索・参照するためのナレッジベースになります。

```bash
# "認証" に関する過去のセッションを検索
grep -l "認証" ~/.claude/Log/Session/*.md

# 特定の日付のセッションを確認
ls ~/.claude/Log/Session/20260118*.md
```

「あの時どうやって解決したっけ？」という時に、過去のセッションから解決方法を見つけられます。

### 2. チーム共有

複雑な実装の経緯をチームメンバーに共有する際に便利です。

- どんな指示を出して
- どんな議論があって
- 最終的にどういう実装になったか

が全て記録されているので、Pull Request のコンテキストとして活用できます。

### 3. 振り返り・学習

AIとのやり取りパターンを分析し、より効果的なプロンプトを発見できます。

- うまくいった指示の特徴
- 手戻りが発生した原因
- より簡潔に伝えられた箇所

## カスタマイズのヒント

### 出力形式の変更

スクリプト内の Markdown 生成部分を修正すれば、好みの形式に変更できます。

例えば、ヘッダー部分を JSON 形式にしたい場合：

```bash
# ヘッダーを JSON 形式で書き込み
cat > "$TEMP_FILE" << EOF
{
  "session_id": "${SESSION_ID}",
  "project": "${PROJECT_PATH}",
  "saved_at": "${DATE_FORMATTED}",
  "trigger": "${TRIGGER}"
}

---
EOF
```

### 保存先の変更

`BASE_DIR` や `LOG_DIR` を変更すれば、任意の場所に保存できます。

```bash
# 例: プロジェクトごとに保存
LOG_DIR="${CLAUDE_PROJECT_DIR:-$PWD}/docs/session-logs"
```

### 整形済みドキュメントの作成

生のセッションログをさらに整形したい場合は、Claude Code のカスタムコマンドと組み合わせることも可能です。例えば `/save-progress` のようなコマンドを作成し、セッションログを要約・整形するワークフローを構築できます。

## まとめ

PreCompact フックを使えば、Claude Code の Compaction による会話履歴の消失を防ぎ、セッション履歴をナレッジとして活用できます。

**設定のポイント**
1. `~/.claude/hooks/pre-compact-log.sh` にスクリプトを配置
2. `~/.claude/settings.json` に PreCompact フックを登録
3. 以後は自動で会話履歴が保存される

一度設定してしまえば、あとは普通に Claude Code を使うだけ。Compaction が発生するたびに、自動で会話履歴が Markdown ファイルとして保存されていきます。

「あの時の議論を見返したい」「チームに共有したい」「プロンプトを改善したい」といった場面で、きっと役に立つはずです。

---

:::message
**関連リソース**
- [Claude Code 公式ドキュメント - Hooks](https://docs.anthropic.com/en/docs/claude-code/hooks)
- [jq 公式サイト](https://jqlang.github.io/jq/)
:::
