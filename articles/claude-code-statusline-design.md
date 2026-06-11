---
title: Claude Code のステータスラインを「見やすさ優先」で育てる — effort 表示と実 JSON ダンプの勘所
emoji: 🐸
type: tech
topics:
  - claudecode
  - ai
  - bash
  - cli
  - jq
published: true
---
![ステータスライン完成イメージ](/images/claude-code-statusline-design/01.png)

## はじめに

Claude Code のステータスライン（status line）は、`reasoning effort` や `thinking`・`fast mode` といった「いまのセッションの状態」を出すと、地味に効きます。effort を上げたつもりで上がっていなかった、thinking を切ったのを忘れていた、といった「気づかない取り違え」が画面の下を見るだけで分かるからです。

ただし情報を足すほど 1 行は散らかります。この記事のテーマは、その両立です。

- 情報は盛る（effort / thinking / fast / トークン / 料金 / コンテキスト使用率 …）
- それでも見やすさは落とさない

まず完成形を見せ、すぐ使えるコピペ一式を置きます。そのあとで「なぜこの形なのか」という設計判断と、フィールド名の確認手順（実 JSON のダンプ）を紹介します。仕組みに興味がない方は、コピペの節までで十分動きます。

## 完成形

これが最終的なステータスラインです。

![完成形のステータスライン](/images/claude-code-statusline-design/02.png)

1 行の内訳はこうなっています（上のスクリーンショットの実際の出力です）。

```
in:114.5k/out:1.7k $2.02 | ctx:11% | Opus 4.8 (1M), high, think | ~/.claude | main | sid:97f7c684-b955-4e9c-a3bd-72a23665743e
```

| 表示 | 意味 |
| --- | --- |
| `in:114.5k/out:1.7k $2.02` | 入力/出力トークンと概算料金 |
| `ctx:11%` | コンテキストウィンドウ使用率 |
| `Opus 4.8 (1M), high, think` | モデル名 / effort / thinking（fast mode はオフなので非表示） |
| `~/.claude` | カレントディレクトリ |
| `main` | git ブランチ |
| `sid:97f7c684-...` | セッション ID（横断参照・ログ特定用にフル表示） |

セッション開始時刻が取れるときは、先頭に `1m23s` のような経過時間も付きます（このスクリーンショットでは出ていません）。

ポイントは「モデルまわり」のグループです。モデル名・effort・thinking（オンのときは fast も）を `, ` でひとかたまりにし、ブロック間は ` | ` で区切る。区切りを 2 階層にするだけで、情報量が増えても「どこからどこまでが一塊か」が読み取れます。`think` と `fast` は有効なときだけ出すので、このスクリーンショットのように fast がオフなら自然に短くなります。

## コピペ用：まず動くものを

仕組みは後回しでまず動かしたい方向けに、再現プロンプトとスクリプト全文を置きます。
この記事のものとは見た目がちょっと異なるものができるかもしれませんが、再現プロントで設定するのが楽だと思います。

### 再現プロンプト

Claude Code にそのまま貼れるプロンプトです。1 番の「実 JSON の確認」を入れておくと、フィールド名の推測ミスを防げます（手順は後述）。

```text
Claude Code のステータスラインを設定したい。次の手順で進めて。

1. まず statusLine スクリプトに渡される stdin の JSON を /tmp にダンプして、
   実際に来るフィールド名を確認して（推測で書かない）。
2. 確認したフィールドを使って、~/.claude/statusline-command.sh を作成し、
   settings.json の statusLine に登録して。表示したいのは次の通り:
   - 経過時間
   - トークン in:/out: と概算料金（0.01 ドル未満は <$0.01 と表示）
   - コンテキスト使用率
   - モデル名（短縮）, reasoning effort, thinking が ON のときだけ think,
     fast mode が ON のときだけ fast（この 4 つは ", " 区切りで 1 グループ）
   - カレントディレクトリ / git ブランチ / セッション ID
   - グループ間は " | " 区切り
3. effort 欠落・think/fast の ON/OFF など各パターンのテスト入力を流して、
   出力が崩れないか確認して。
```

### スクリプト全文

`~/.claude/statusline-command.sh` として保存します（セットアップ手順は下記）。

:::details スクリプト全文を表示（クリックで開閉）

```bash
#!/bin/bash
# Claude Code ステータスライン スクリプト

input=$(cat)

# --- モデル名（判別できる範囲で短縮：" context)" → ")"）---
model=$(echo "$input" | jq -r '.model.display_name // "Claude"')
model="${model/ context)/)}"

# --- reasoning effort / thinking / fast（1 回の jq でまとめて取得）---
# 1 行 1 値で出力し行単位で read（空値でもフィールドがずれないようにする）
{ read -r effort; read -r thinking; read -r fast_mode; } < <(
  echo "$input" | jq -r '.effort.level // "", (.thinking.enabled // false | tostring), (.fast_mode // false | tostring)'
)

# --- セッション ID（横断参照・ログ特定用にフル表示）---
session_id=$(echo "$input" | jq -r '.session_id // empty')

# --- 現在のディレクトリ（ホームを ~ に短縮）---
cwd=$(echo "$input" | jq -r '.workspace.current_dir // .cwd // ""')
cwd_short="${cwd/#$HOME/~}"

# --- Git ブランチ ---
branch=""
if [ -n "$cwd" ] && git -C "$cwd" rev-parse --is-inside-work-tree >/dev/null 2>&1; then
  branch=$(git -C "$cwd" --no-optional-locks symbolic-ref --short HEAD 2>/dev/null)
fi

# --- セッション経過時間（トランスクリプトの最初のエントリのタイムスタンプから算出）---
elapsed=""
transcript=$(echo "$input" | jq -r '.transcript_path // ""')
if [ -n "$transcript" ] && [ -f "$transcript" ]; then
  first_ts=$(head -1 "$transcript" 2>/dev/null | jq -r '.timestamp // empty' 2>/dev/null)
  if [ -n "$first_ts" ]; then
    start_epoch=$(date -j -f "%Y-%m-%dT%H:%M:%S" "${first_ts%%.*}" "+%s" 2>/dev/null \
                  || date -d "${first_ts%%.*}" "+%s" 2>/dev/null)
    now_epoch=$(date "+%s")
    if [ -n "$start_epoch" ] && [ -n "$now_epoch" ]; then
      diff_sec=$(( now_epoch - start_epoch ))
      if [ $diff_sec -lt 0 ]; then diff_sec=0; fi
      h=$(( diff_sec / 3600 )); m=$(( (diff_sec % 3600) / 60 )); s=$(( diff_sec % 60 ))
      if [ $h -gt 0 ]; then elapsed=$(printf "%dh%02dm" $h $m)
      else elapsed=$(printf "%dm%02ds" $m $s); fi
    fi
  fi
fi

# --- トークン数と料金（累計）---
total_in=$(echo "$input"  | jq -r '.context_window.total_input_tokens  // 0')
total_out=$(echo "$input" | jq -r '.context_window.total_output_tokens // 0')

# claude-opus-4-x 系の概算単価（入力 $15/M、出力 $75/M、キャッシュ読み $1.5/M、キャッシュ書き $18.75/M）
cache_read=$(echo "$input"  | jq -r '.context_window.current_usage.cache_read_input_tokens  // 0')
cache_write=$(echo "$input" | jq -r '.context_window.current_usage.cache_creation_input_tokens // 0')

cost=$(echo "$total_in $total_out $cache_read $cache_write" | awk '{
  cost = ($1 * 15 + $2 * 75 + $3 * 1.5 + $4 * 18.75) / 1000000
  if (cost > 0 && cost < 0.01) { printf "<$0.01" } else { printf "$%.2f", cost }
}')

# トークン表示（1000 単位で k 表示）
fmt_tokens() {
  local n=$1
  if [ "$n" -ge 1000 ] 2>/dev/null; then
    echo "$(echo "$n" | awk '{printf "%.1fk", $1/1000}')"
  else
    echo "$n"
  fi
}
tok_in=$(fmt_tokens "$total_in")
tok_out=$(fmt_tokens "$total_out")

# --- コンテキスト使用率 ---
ctx_pct=$(echo "$input" | jq -r '.context_window.used_percentage // empty')
ctx_str=""
[ -n "$ctx_pct" ] && ctx_str=$(printf "ctx:%.0f%%" "$ctx_pct")

# --- 組み立て ---
# 配列を区切り文字で連結するヘルパー（join_by SEP ELEM...）
join_by() {
  local sep="$1"; shift
  local out=""
  for x in "$@"; do
    if [ -z "$out" ]; then out="$x"; else out="${out}${sep}${x}"; fi
  done
  printf '%s' "$out"
}

parts=()
[ -n "$elapsed" ] && parts+=("$elapsed")
parts+=("in:${tok_in}/out:${tok_out} ${cost}")
[ -n "$ctx_str" ] && parts+=("$ctx_str")

# モデル / effort / think / fast（カンマ区切りで 1 グループ）
mgroup=()
[ -n "$model" ] && mgroup+=("$model")
[ -n "$effort" ] && mgroup+=("$effort")
[ "$thinking" = "true" ] && mgroup+=("think")
[ "$fast_mode" = "true" ] && mgroup+=("fast")
[ ${#mgroup[@]} -gt 0 ] && parts+=("$(join_by ", " "${mgroup[@]}")")

[ -n "$cwd_short" ] && parts+=("$cwd_short")
[ -n "$branch" ] && parts+=("$branch")
[ -n "$session_id" ] && parts+=("sid:$session_id")

result=$(join_by " | " "${parts[@]}")
echo "$result"
```

:::

### セットアップ手順

要点だけ。

1. 上のスクリプトを `~/.claude/statusline-command.sh` に保存
2. `jq` を導入（未導入時）: `brew install jq`（macOS）/ `sudo apt install jq`（Linux）
3. 動作確認（任意）: 下のサンプル JSON を流して 1 行出れば OK
4. `~/.claude/settings.json` に下の `statusLine` ブロックを追加（パスは自分のホームに合わせる）
5. 反映: スクリプトの変更は次の描画から即時反映（再起動不要）。`statusLine` を新規追加したときはセッションを開き直すと確実

動作確認（手順3）:

```bash
echo '{"model":{"display_name":"Opus 4.8 (1M context)"},"context_window":{"total_input_tokens":1000,"total_output_tokens":50,"used_percentage":5,"current_usage":{}},"effort":{"level":"high"}}' | bash ~/.claude/statusline-command.sh
# => in:1.0k/out:50 $0.02 | ctx:5% | Opus 4.8 (1M), high
```

settings.json への登録（手順4）:

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash /Users/<you>/.claude/statusline-command.sh"
  }
}
```

補足:

- `command` を `bash …` で起動しているので実行権限の付与（`chmod +x`）は不要。スクリプトをパス直指定で呼ぶ場合のみ `chmod +x` する。
- 一から作らせるなら `/statusline` に自然言語で指示すれば、スクリプト生成と settings.json への登録まで自動でやってくれる。確実性を上げたいときは「まず実 JSON をダンプして確認してから」と添えるとよい（手順は後述）。

ここまでで動きます。料金はあくまで概算（単価ハードコード）です。正確な値の出し方は最後の「さらに良くする」で触れます。仕組みと設計判断が気になる方は、このまま読み進めてください。

## どう値が来るのか（最小限）

ステータスラインの仕組みは 1 枚で説明できます。

1. `settings.json` の `statusLine` を `type: "command"` にすると、Claude Code が指定スクリプトを実行する
2. そのとき、セッションの状態が **JSON で標準入力（stdin）に流れてくる**
3. スクリプト側は `jq` でほしいフィールドを抜き出し、整形して 1 行を `echo` する

つまり「JSON を受け取って、好きに整形して 1 行返す」だけ。あとは、どのフィールドが来るかを確認すれば書けます。

## フィールド名は実 JSON で確認する

フィールド名は推測で書くと間違えやすく、間違っても `jq` は空文字を返すだけなので気づきにくいです（たとえば effort の正しいパスは `.effort.level`。`.reasoning_effort` と書いても何も出ないだけ）。作る前に一度、実物をダンプして確認しておきます。スクリプトの先頭に 1 行足すだけです。

```bash
input=$(cat)
echo "$input" > /tmp/statusline-input.json   # ← デバッグ用。確認できたら消す
```

ステータスラインは数百ミリ秒ごとに再評価されるので、数秒待てば `/tmp` にダンプされます。キー一覧を見てみます。

```bash
jq -r 'keys[]' /tmp/statusline-input.json
```

```
context_window
cost
cwd
effort
exceeds_200k_tokens
fast_mode
model
output_style
rate_limits
session_id
session_name
thinking
transcript_path
version
workspace
```

`effort`・`thinking`・`fast_mode` が実在することが分かります。中身も見ます。

```bash
jq -c '.effort, .thinking, .fast_mode' /tmp/statusline-input.json
# {"level":"high"}
# {"enabled":true}
# false
```

`.effort.level` / `.thinking.enabled` / `.fast_mode` を使えばよい、と確定できました。

### 公式に載っているもの・いないもの

2026-06-09 現在の公式ドキュメント（Claude Code の Status line ページ）と照らし合わせると、差があります。

| フィールド | 公式ドキュメント | 補足 |
| --- | --- | --- |
| `.effort.level` | 記載あり | `low` / `medium` / `high` / `xhigh` / `max`。`/effort` でのセッション中の変更も即反映。effort 非対応モデルでは欠落 |
| `.thinking.enabled` | 記載あり | 拡張思考の ON/OFF |
| `.fast_mode` | 記載なし | ドキュメントには無いが、実際の JSON には存在する |

`effort` と `thinking` は公式にあるのでプロンプトで頼んでも当たりやすい一方、`fast_mode` のような未ドキュメントのフィールドは、ダンプして実物を見ないと拾い損ねることがあります。

## 見やすさを保つ設計判断

情報を盛りつつ散らからせないために、3 つの判断をしています。

### 1. 区切りを 2 階層にする

すべてを ` | ` で並べると、関連の強い項目（モデル・effort・thinking・fast）まで横並びになり、視線が滑ります。そこで、関連する 4 つは `, ` でまとめて 1 グループにし、グループ間だけ ` | ` で区切ります。

```
... | Opus 4.8 (1M), high, think, fast | ~/.claude | main | ...
```

「モデルまわり」が一塊に見えるだけで、ぐっと読みやすくなります。

### 2. 出すべきときだけ出す（条件表示）

`think` と `fast` は ON のときだけ表示します。常に `think:off` のように状態を出すと、情報としては正確でも、9 割の時間「いつも同じ文字」が場所を取るだけです。バッジは「いつもと違うときに気づける」ことに価値があるので、オフのときは消します。

```
think+fast ON : ... | Opus 4.8 (1M), high, think, fast | ...
両方 OFF      : ... | Opus 4.8 (1M), high | ...
```

### 3. 数値はノイズを削る

- 料金は小数 2 桁（`$0.97`）。ただし 0.01 ドル未満は `<$0.01` にして、序盤に `$0.00` が並ぶのを避ける
- トークンは `in:59.6k/out:978` のように `in:`/`out:` を前置きし、1000 以上だけ `k` 表記
- モデル名は `Opus 4.8 (1M context)` を `Opus 4.8 (1M)` に短縮（判別はできる範囲で）

どれも小さな調整ですが、合わさると 1 行の密度が一段読みやすくなります。

## リファクタの一幕：AI に直させ、検証で回帰を捕まえる

ここからは少し実装寄りの話です。育てる過程で、AI 駆動ならではの「型」が出たので残しておきます。

### 連結ロジックの重複を 1 つに

グループを `, ` でつなぐループと、最終的に全体を ` | ` でつなぐループは、区切り文字が違うだけで同じ処理でした。`join_by` という小さなヘルパーに集約して、2 か所から呼ぶ形にしています。区切りを引数にするだけで重複が消えます。

```bash
join_by() {        # join_by SEP ELEM...
  local sep="$1"; shift
  local out=""
  for x in "$@"; do
    if [ -z "$out" ]; then out="$x"; else out="${out}${sep}${x}"; fi
  done
  printf '%s' "$out"
}
```

### jq 呼び出しの本数を減らす

effort・thinking・fast を別々の `jq` で 3 回読んでいたのを、1 回にまとめました。ステータスラインは毎回再評価される hot path なので、サブプロセスの数は少ないほうが良いです。

### IFS の落とし穴（ここがいちばんの学び）

3 つを 1 回で取るとき、最初は `@tsv`（タブ区切り）でまとめて `read` しようとしました。

```bash
# これは罠
IFS=$'\t' read -r effort thinking fast_mode < <(
  echo "$input" | jq -r '[.effort.level // "", .thinking.enabled, .fast_mode] | @tsv'
)
```

テストで effort が欠落するモデルを流したら、`Sonnet 4.6, false` という妙な表示が出ました。effort に `false` が入っている。

原因は `IFS`。タブは「IFS 空白文字」の一種なので、先頭が空フィールド（`\tfalse\tfalse`）だと **先頭の空白が trim され、値が 1 つ左にずれます**。effort が空のはずなのに、thinking の `false` が effort に入ってしまったわけです。

対策は単純で、タブ 1 行に詰めず、**1 行 1 値で出して行単位で `read`** します。空行はそのまま空文字として読まれるので、ずれません。

```bash
{ read -r effort; read -r thinking; read -r fast_mode; } < <(
  echo "$input" | jq -r '.effort.level // "", (.thinking.enabled // false | tostring), (.fast_mode // false | tostring)'
)
```

この回帰は「AI に整理させた直後にテストを流して」見つかりました。AI に任せる範囲を広げるほど、こうした「もっともらしいが間違い」を**自分でテストして捕まえる**工程が効いてきます。出力が崩れやすいのは、effort 欠落・think/fast の ON/OFF あたりなので、最低この 4 パターンは流すのがおすすめです。

## プロンプトだけで作れるのか

作れます。経路は 2 つあります。

1. **`/statusline`（ビルトイン）** — 自然言語で「こう表示して」と渡すと、Claude Code が `~/.claude/` にスクリプトを生成し、`settings.json` も自動更新します（削除は `/statusline delete`）。ファイルを手で触る必要はありません
2. **通常の会話プロンプト** — この記事のように、対話で少しずつ整える

どちらもプロンプトだけで完結します。最初に貼った再現プロンプトのように「まず実 JSON をダンプして確認させる」一手を含めておくと、未ドキュメントの `fast_mode` のようなフィールドも取りこぼしにくくなります。

## さらに良くする

最後に、この記事のスクリプトから一段進める余地を 2 つ。

### 料金は公式の値を使える

このスクリプトは料金をトークン数 × 単価で手計算していますが、実は JSON に **`.cost.total_cost_usd`** が来ています。これを使えば、単価のハードコードも、モデルごとの単価差も気にせず、正確な値が出せます。

```bash
cost=$(echo "$input" | jq -r '.cost.total_cost_usd // 0 | "$" + (. * 100 | round / 100 | tostring)')
```

「自前計算 → 公式値」に置き換えると、メンテも軽くなります（学習目的で計算式を見せたいとき以外は、公式値で十分です）。

### 他にも使えるフィールドがある

ダンプして眺めると、ほかにも表示に使える情報が見つかります。

- `rate_limits`（5 時間 / 7 日のレート上限の使用率・リセット時刻）
- `pr`（PR 番号・URL・レビュー状態）
- `vim.mode`（Vim モード）
- `workspace.repo`（git origin の host / owner / name）
- `output_style`（出力スタイル名）

自分のワークフローで「画面の下にあると嬉しいもの」を選んで足していけば、ステータスラインは自分専用のダッシュボードになります。

## まとめ

「見やすさを保ったまま情報を盛る」ための原則は、3 つに圧縮できます。

1. **グループ化** — 区切りを 2 階層にして、関連項目を一塊に見せる
2. **条件表示** — 常に出さず、いつもと違うときだけ出す
3. **実物で確認** — フィールドは推測せず、実 JSON をダンプして確定する

3 つめは地味ですが、説明して当てにいくより一度ダンプして見るほうが結局速いです。
