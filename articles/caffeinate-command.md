---
title: "フタを閉じても MacBook Pro をスリープさせない。`caffeinate` コマンド まとめ"
emoji: "🐸"
type: "tech"
topics: [mac, terminal, shell, claudecode]
published: true
---

移動中に MacBook Pro を閉じたままでも、「Claude Code などの AIエージェントを止めずに走らせ続けたい。」、「ファイルのアップロードを続けさせたい」。

そんなときに便利なのが、macOS に標準搭載されている `caffeinate` コマンド。ターミナルから一行入力するだけでスリープを防ぎ、大事な処理を途中で止めることなく継続させることができます。

この記事では、`caffeinate` コマンドの使い方と活用法についてまとめました。

※ 面倒と思われる方は、アプリ [Amphetamine on the Mac App Store](https://apps.apple.com/us/app/amphetamine/id937984704?mt=12) で同様のことができます。

## `caffeinate` コマンドの概要

`caffeinate` は、**macOS を一時的にスリープさせないようにする**ためのコマンドラインユーティリティ。  
特定のプロセス実行中や時間指定で、スリープ・ディスプレイオフ・ディスク休止などを防止できます。  作業中や長時間処理中に Mac がスリープしてしまうのを防ぎたいときに便利です。

コマンドを実行している間だけ効果があり、**中断（Ctrl+C）すると元に戻ります**。

## `caffeinate` コマンドのオプションと使用例

- `-d`: ディスプレイのスリープを防止 (`caffeinate -d`)
- `-i`: アイドルスリープを防止 (`caffeinate -i my_app`)
- `-m`: ディスク休止を防止 (`caffeinate -m diskutil verifyVolume /`)
- `-s`: システム全体のスリープを防止 (`caffeinate -s backup.sh`)
- `-u`: ユーザー操作を擬装 (`caffeinate -u -t 30`)
- `-t N`: N 秒間だけスリープ防止 (`caffeinate -t 600`)
- `-w PID`: 指定 PID 終了まで防止 (`caffeinate -w 1234`)
- `[command]`: コマンド実行中スリープ防止 (`caffeinate -i long_task.sh`)

### 複数オプションの適用

- `-is`: アイドル＋全体スリープを両方防止 (`caffeinate -is`)
- `-iu`: アイドルスリープ防止＋ユーザー操作を擬装 (`caffeinate -iu -t 300`)
- `-dt 600`: ディスプレイを10分間オンに保つ (`caffeinate -dt 600`)

### 補足: `-u`（ユーザー操作の擬装）とは？

`-u` は、macOS に「ユーザーが最近操作した」と見せかける信号を送り、ディスプレイのスリープやスクリーンセーバーの起動を防ぐためのオプションです。単体では約5秒しか効果が続かないため、通常は `-t`（秒数指定）と併用します。プレゼンや動画再生時など、実際に操作していなくても画面を暗くしたくない場面で便利です。

### 補足: プロセスID (pid) の調べ方

“claude" を調べたい時  
`ps aux | grep claude`

---

## 自分用メモ: AIエージェントに実行させるには

★★未確認、要検証

- エージェントにプロンプト `今実行している Claude Code 自身のpid を教えて` で pid を取得させ、`caffeinate -w <取得したPID>` を実行させればいける？ それを Hooks で開始時に自動実行させる？
- その他メモ
  - 監視用スクリプトを用意して、`.claude/config.json` に `{"run": "caffeinate -is ./監視用スクリプト.sh"}` を追記すると、Claude Code 自身を実行する際にスリープを防ぐ？
  - `caffeinate -w $(pgrep -f 'claude.*code')`
