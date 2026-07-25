---
title: "Claude Code のセッションログを macOS の APFS 透過圧縮で削減する"
emoji: "🐸"
type: "tech"
topics:
  - claudecode
  - macos
  - apfs
  - launchd
published: true
---

![書類の山を万力プレスで圧縮する女性と、縮んだファイル束を抱えるオレンジ色のキャラクターのイラスト](/images/claude-code-session-log-apfs-compression/eyecatch.png)

Claude Code を毎日使っていると、`~/.claude/projects/` 配下にセッションログ（会話履歴の JSONL ファイル）が静かに積み上がっていきます。私の環境では、気づいたら合計 2.4GB。一番使っているプロジェクトだけで 436MB・408 ファイルになっていました。

消せば済む話ではあります。ただ、古いセッションログには消したくない理由がある。この記事では、ログを 1 ファイルも消さず、grep も `/resume` もそのまま使える状態を保ったまま、macOS の透過圧縮でディスク消費だけを減らす方法を書きます。

仕組みはファイルシステム側の機能なので、Claude Code 専用ではありません。会話ログを JSONL で溜め込む他の AI エージェントにも同じ手が使えます。実際、私の環境では Codex CLI の `~/.codex/sessions` が 7.5GB ありました。こちらにも効きます。（ただし Codex では今回の方法を検証していません。Codex のセッションログにはセッション中に生成した画像のデータが埋め込まれるため、画像生成を多用してきた私の環境には 1GB 近いログもあります。この規模になると多少のファイル削減では対処しきれないので、別の方法を模索中です…）

:::message alert
この方法は **macOS 専用**です（APFS ファイルシステムの機能を使うため）。Windows / Linux の Claude Code には適用できません。

**また、本記事の内容は無保証です。実施した結果生じたいかなる損害についても、筆者は責任を負えません。バックアップがある状態で、自己責任で試してください。**

検証環境: macOS 15（Sequoia）/ afsctool 1.7.3 / Claude Code。
:::

## TL;DR

- 30 日以上前のセッション JSONL 3,474 本を APFS 透過圧縮したら、`~/.claude/projects` 全体が **2.4GB → 1.8GB** になった
- 圧縮後も **grep・`/resume`・アプリからの読み込みはすべて無変更で動く**
- launchd で月 1 回の自動実行にして、以後は放置で節約が維持される

## AI エージェントと会話を重ねるごとに膨らむセッションログ

Claude Code は全会話を `~/.claude/projects/<プロジェクト別フォルダ>/<セッションID>.jsonl` に永続化します。入っているのは会話本文だけではありません。ツールの実行結果 ─ 読み込んだファイルの中身、コマンドの出力 ─ も丸ごと記録されるため、長めの作業セッションは 1 ファイルで 10MB を超えます。さらにサブエージェントを使えば、そのログも別ファイルで残ります。

数ヶ月使い込んだ私の環境の実測がこうでした。

```
$ du -sh ~/.claude/projects
2.4G    /Users/xxx/.claude/projects
```

放っておけば増える一方です。

## それでも、古いセッションは消さない方がいい

削除をためらったのには理由があります。古いセッションログは、ただのゴミではなく資産だからです。

- **過去の解決策を grep で発掘できる。** 「このエラー、前にどう直したっけ」が、ログへの全文検索一発で返ってきます。メモに残し損ねた知見の最後の砦です
- **`/resume` で作業を再開できる。** 数週間寝かせた案件の続きを、当時の文脈ごと引き継げます
- **AI が何をしたかの一次記録になる。** 意図しない編集が起きたとき、いつ・何を実行したかを遡れる唯一の証跡です
- **消えたデータの復元経路になる。** 上書き・削除してしまったファイルの旧内容が、ツール実行結果の中に残っていることがあります

Claude Code はこのログを既定で 30 日しか保持せず、それを過ぎた分は自動削除されます。私は残したい派なので、`settings.json` に `"cleanupPeriodDays": 1095` を設定して保持期間を 3 年に延ばしています。

なので残せるものなら全部残しときたい。でもディスク容量は膨らんでいく。ここで「圧縮すればいいのでは」となるわけですが、gzip 圧縮はよく縮むもののアーカイブであって、使える状態のまま小さくする手段ではありません。grep が直接効かなくなるし、`/resume` もできなくなります。

欲しいのは、ファイル名も中身も見かけ上そのまま、ディスク上の実体だけが縮む圧縮。そんな都合の良いものがあるのか？… あります。「APFS 透過圧縮」です。

## APFS 透過圧縮とは

macOS には decmpfs と呼ばれる透過圧縮の仕組みが Snow Leopard（2009 年、懐かしい）の時代からあり、OS 自身がシステムファイルやアプリケーションで常用しています。圧縮データはリソースフォーク・拡張属性側に格納され、読み取り時にはカーネルが自動で解凍する。だからアプリからは、圧縮前とまったく同じ普通のファイルに見えます。grep も cat も Claude Code も、圧縮されていることに気づきません。

ただし macOS は、ユーザーのファイルを勝手に圧縮してはくれません。そこで [afsctool](https://github.com/RJVB/afsctool) というオープンソースの CLI を使って、任意のファイルに後付けします。

使う前に知っておくべき性質が 1 つあります。**圧縮済みファイルに書き込むと、自動で非圧縮状態に戻ります。** データは無傷ですが、節約分は消える。古いセッションを `/resume` して続きを書いた場合がこれに当たります。後述の launchd 自動化は、この取りこぼしを月次で回収するための仕掛けです。

## 設定手順

### 1. afsctool のインストール

```bash
brew install afsctool
```

### 2. まず 1 ファイルで試す

いきなり全体にかけず、古いログを 1 つ選んで挙動を確かめます。

```bash
# 30 日以上更新のない、大きめのログを探す
find ~/.claude/projects -name "*.jsonl" -mtime +30 -size +1M | head -3

# 圧縮前のチェックサムを控えておく（検証用）
shasum -a 256 <対象ファイル>

# 圧縮する
afsctool -c <対象ファイル>

# 圧縮状態と節約量を確認
afsctool -lv <対象ファイル>

# チェックサムが一致し、普通に grep できることを確認
shasum -a 256 <対象ファイル>
grep -c '"type"' <対象ファイル>
```

私の環境では 11MB のログが 7.5MB になり（節約 34.6%）、SHA-256 チェックサムは圧縮前後で完全一致。grep もそのまま動きました。

:::message alert
`-mtime +30` のような「古いファイルだけ」の条件を必ず付けてください。書き込み中の当日セッションに触れるのは避けます。
:::

### 3. 全体に適用する

試験で問題なければ、30 日超の全ログに一括適用します。`-j4` は 4 並列の指定です。

```bash
find ~/.claude/projects -name "*.jsonl" -mtime +30 -exec afsctool -c -j4 {} +
```

## 実測結果

| 対象 | 圧縮前 | 圧縮後 |
|---|---|---|
| `~/.claude/projects` 全体 | 2.4GB | **1.8GB** |
| 11MB のセッションログ | 11MB | 7.5MB（−34.6%） |
| 8.6MB のセッションログ | 8.6MB | 5.2MB（−39%） |
| 21KB のサブエージェントログ | 21KB | 8KB（−61.5%） |

対象は 3,474 ファイル。大きなログはおおむね 3 分の 1 強縮み、小さなログには 6 割縮むものもありました。

順調に見えますが、実行中に 1 つ焦る場面がありました。並列実行の途中で `Unable to compress file ...` というメッセージが大量に流れたんです。失敗かと思って表示されたファイルを個別に確認すると ─ 実際には圧縮に成功していて、節約 61.5%。全ファイルをフラグで集計しても未圧縮はゼロでした。少なくとも並列実行時のこのメッセージは、実態を反映していません。成否は表示ではなく、ファイルの状態で判定します。

```bash
# compressed と出れば圧縮済み
stat -f %Sf <ファイル>
```

もう 1 つ、ZLIB の圧縮レベルを既定の 5（`afsctool -h` に default is 5 と明記されています。zlib ライブラリ一般の既定 6 とは別です）から 9 に上げても削減量は変わりませんでした。中間レベルで既にこのデータの圧縮限界に達しているようで、既定のままで問題ありません。

## launchd で自動化する

前述のとおり、`/resume` で追記されたログは非圧縮に戻ります。手動で再実行し続けるのは現実的でないので、launchd で毎月 1 日の朝に自動実行させます。指定時刻にスリープ中だった場合は、次の起床時に実行されます。

```xml:~/Library/LaunchAgents/com.user.claude-session-compress.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>com.user.claude-session-compress</string>
	<key>ProgramArguments</key>
	<array>
		<string>/bin/sh</string>
		<string>-c</string>
		<string>echo "=== $(date '+%Y-%m-%d %H:%M:%S') 開始 ==="; /usr/bin/find "$HOME/.claude/projects" -name "*.jsonl" -mtime +30 -exec /opt/homebrew/bin/afsctool -c -j4 {} + ; echo "=== 終了 (exit $?) ==="</string>
	</array>
	<key>StartCalendarInterval</key>
	<dict>
		<key>Day</key>
		<integer>1</integer>
		<key>Hour</key>
		<integer>7</integer>
		<key>Minute</key>
		<integer>30</integer>
	</dict>
	<key>StandardOutPath</key>
	<string>/Users/あなたのユーザー名/Library/Logs/claude-session-compress.log</string>
	<key>StandardErrorPath</key>
	<string>/Users/あなたのユーザー名/Library/Logs/claude-session-compress.log</string>
	<key>ProcessType</key>
	<string>Background</string>
	<key>LowPriorityIO</key>
	<true/>
	<key>Nice</key>
	<integer>10</integer>
</dict>
</plist>
```

`StandardOutPath` / `StandardErrorPath` の `あなたのユーザー名` は実際のユーザー名に書き換えてください（launchd のログ出力先は絶対パスで書く必要があります）。実行タスクは低優先 I/O と nice 10 を付けているので、朝の作業と重なっても邪魔になりません。

登録と動作確認はこうです。

```bash
# 構文チェックと登録
plutil -lint ~/Library/LaunchAgents/com.user.claude-session-compress.plist
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.user.claude-session-compress.plist

# その場でテスト実行して、ログを確認
launchctl kickstart gui/$(id -u)/com.user.claude-session-compress
tail ~/Library/Logs/claude-session-compress.log
```

テスト実行のログに `No compressable files found.` と出れば正常です（直前に手動で全体適用した直後なら、圧縮対象が残っていないため）。やめたくなったら `launchctl bootout gui/$(id -u)/com.user.claude-session-compress` で解除できます。

## 気になるであろう質問

### ファイルが壊れたりしない？

afsctool は既定で、圧縮後にファイルの検証を行います（`-n` で省略できますが、ヘルプ内でも非推奨扱いです）。私も全ファイルで SHA-256 の一致を確認しました。

ただ、decmpfs をユーザーファイルに使うことを Apple は公式にドキュメント化していません。afsctool も個人メンテナンスのオープンソースです。私が全体適用に踏み切ったのは、対象が「最悪失われても、そのセッションを `/resume` できなくなるだけのログ」だったから。例えば仕事の成果物そのものに使う場合は、より慎重に判断してください。

### 圧縮したファイルに書き込んだら？

自動で非圧縮に戻ります。もちろん、データは無傷です。上記の launchd による自動化を設定しておけば、非圧縮に戻っても翌月の launchd 実行が再圧縮します。

### テキストなのに 3 割しか縮まないの？

decmpfs の制約ではなく、データ側の圧縮限界です。同じログを gzip -9 でファイル全体を一括圧縮しても削減率は 35.4% で、decmpfs の 34.6% とほぼ同じでした。セッションログには圧縮の効きにくいデータ（Base64 エンコードされた画像など）が混ざるため、テキストの見た目ほどは縮みません。圧縮レベルを 9 に上げても変わらなかったので、この用途では 3〜4 割が相場だと受け止めています。

### Time Machine やクラウド同期との相性は？

バックアップ・同期ツールがファイルを読むときは解凍後の中身が流れるので、**節約が効くのはローカルディスクだけ**です。壊れるという報告は調べた範囲では見つかりませんでしたが、相互作用を保証する公式情報もありません。この点は未検証です。

### `ls` で見てもサイズが減っていないけど？

仕様です。`ls -l` が表示するのは論理サイズ（解凍後の大きさ）で、実際のディスク消費は `du` で確認します。

```bash
# ls は圧縮前と同じサイズを表示する（論理サイズ）
$ ls -lh <セッションID>.jsonl
-rw-------  1 xxx  staff    11M  6 14 16:22 <セッションID>.jsonl

# du は実際のディスク消費を表示する（圧縮後）
$ du -h <セッションID>.jsonl
7.5M    <セッションID>.jsonl
```

この 2 つが乖離していたら、透過圧縮が効いている証拠です。

### やめたくなったら？

`afsctool -d <ファイル>` で元の状態に戻せます。

## まとめ

古いセッションログの扱いは「消して容量を取るか、残して容量を諦めるか」の二択だと思っていました。APFS 透過圧縮を知ってからは、その二択から降りられた。3,474 ファイルを圧縮して 600MB 減、grep も `/resume` も一切犠牲にしていません。

セットアップは `brew install afsctool` と find の 1 行、あとは launchd に任せるだけです。あなたの環境でどれくらい縮んだか、X（[@nagataku_ai](https://x.com/nagataku_ai)）で教えてもらえると嬉しいです。
