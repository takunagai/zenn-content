---
title: 【Claude Code】作業中の AI に介入する steer・queue・中断の正確な使い分け
emoji: 🐸
type: tech
topics:
  - claudecode
  - cli
  - ai
  - anthropic
published: true
---
![介入キー早見表](/images/claude-code-intervention-keys/01.png)

## この記事の対象

Claude Code が作業している最中に、こちらから「止める」「方向を直す」「あとで指示を足す」介入操作を、**早見表とキー別の短い解説**でまとめます。すべて公式ドキュメントの記述に基づきます。

対象は、Claude Code を日常的に使っていて、こんな場面で手が止まる人です。

- 止めたいが `Esc` と `Ctrl+C` のどちらが正しいのか毎回迷う
- 作業を止めずに指示を足したいが、入力していいのか分からない
- 書きかけの入力を失わずに、別の指示を先に送りたい

検証環境は 2026 年 6 月時点、Claude Code CLI の v2.1 系（手元は `v2.1.177`）です。挙動は変わることがあるので、手元のバージョンは `claude --version` で確認してください。

## 進行コントロール ショートカット一覧（早見表）

まず全体像です。以降は、この表の上から順に各キーを解説します。

| キー                             | 公式の説明                                        | 役割             |
| ------------------------------ | -------------------------------------------- | -------------- |
| `Enter`（作業中に入力後）               | 実行中のツールを止めずに送信。次の手を決める前に反映                   | ソフトな steer     |
| `↑`（入力欄が空のとき）                  | 直近に queue したメッセージを入力欄へ取り出す（編集・取り消しできる）       | 最新キューの呼び戻し     |
| `↑` / `↓`（`Ctrl+P` / `Ctrl+N`） | カーソル移動、端まで来たらコマンド履歴をたどる                      | 送信前入力の呼び戻し     |
| `Esc`                          | 現在の応答・ツール呼び出しを中断。それまでの作業は保持                  | ハードな割り込み       |
| `Ctrl+C`                       | 実行中は中断。何も動いていなければ入力クリア、2回で終了                 | 中断・終了寄り        |
| `Esc` `Esc`                    | 入力があれば下書きを消す／空なら rewind メニュー                 | やり直し・巻き戻し      |
| `Ctrl+S`                       | 書きかけのプロンプトを一時退避。別の送信後に自動復帰                   | 下書きの棚上げ（stash） |
| `Shift+Tab`                    | パーミッションモードを循環（default→acceptEdits→plan→auto） | 実行権限の制御        |
| `Ctrl+B`                       | 実行中の bash・エージェントをバックグラウンド化（tmux は2回）         | 退避・並行実行        |
| `Ctrl+X` `Ctrl+K`              | このセッションのバックグラウンドサブエージェントを全停止（3秒以内に2回）        | 一括停止           |
| `Ctrl+T`                       | タスクリストの表示切り替え                                | 進捗監視           |
| `/btw`                         | 作業を止めずにサイド質問（ツールなし・履歴に残らない）                  | 非破壊の割り込み質問     |

出典は公式ドキュメントの "Interactive mode" / "How Claude Code works" / "Customize keyboard shortcuts" です（末尾にリンク）。

## Enter：止めずに舵を切る（steer）

いちばん誤解されやすい点から。**作業中に入力して `Enter` を押しても、実行中のツールは止まりません。** 公式 "How Claude Code works" の「Interrupt and steer」にこうあります。

> Type a correction and press `Enter` to send it without stopping the running tool. Claude reads it as soon as the current action completes and adjusts before deciding its next step.
> (修正内容を入力して `Enter` を押すと、実行中のツールを停止することなく送信できます。現在の処理が完了し次第 Claude がそれを読み込み、次のステップを決定する前に調整を行います。)

送ったメッセージは実行を止めず、いま動いているアクションが終わった直後・次の手を決める前に読まれて反映されます。これが **steer（舵取り）** です。

- **向く**：軌道は大きく外れていないが、方向を少し足したい・絞りたいとき
- **向かない**：いま動いているツール自体を止めたいとき（それは後述の `Esc`）

## queue：指示を積む・呼び戻す・取り消す

作業中に続けて `Enter` で送ると、メッセージは順番に処理待ちへ積まれます（複数可）。

積んだうち**最新の1件は、入力欄が空の状態で `↑` を押すと入力欄に戻せます**（queue から取り出されるので、編集して送り直すことも、そのまま消すこともできます）。

ただし、**途中の1件だけを個別に編集・並べ替え・取り消すUIはありません。** 古い1件を抜きたいときは、`Esc` か `Ctrl+C` で全体を止めて入れ直すことになります。この不便さには要望が続いていて、2026 年 6 月時点で [#34835](https://github.com/anthropics/claude-code/issues/34835)（キュー機能全般の本体）ほか複数が open です。

## Esc / Ctrl+C / Esc Esc：止める・巻き戻す(rewind)

「止める」に見える3操作は役割が違います。

**`Esc`：止めて保持** — 実行中の応答・ツールをその場で止め、入力待ちに戻します。

> Stop the current response or tool call mid-turn so you can redirect. Claude keeps the work done so far
> (進行中の回答やツール呼び出しを途中で停止して、方向修正を行うことができます。それまでに完了した作業はそのまま保持されます。)

「それまでの作業は保持される」のがポイントです。ゼロからのやり直しではなく「一度止めて指示し直す」操作です。

**`Ctrl+C`：中断・終了寄り** — 実行中は中断します。何も動いていないときは1回目で入力クリア、2回目で Claude Code を終了します。「抜ける」方向に倒れているキーなので、作業を続けたいなら `Esc` が安全です。

**`Esc` `Esc`：下書きクリア／巻き戻し** — 入力欄に文字があれば下書きを消し（`↑` で復元可）、空なら rewind メニューが開いて、コードと会話を過去の地点に復元できます。

> When the prompt input contains text, double `Esc` clears it and saves the draft to history so `Up` recalls it. When the input is empty, double `Esc` opens the rewind menu to restore or summarize code and conversation from a previous point
> (プロンプト入力欄にテキストがある場合、`Esc` を2回押すとテキストが消去され、下書きが履歴に保存されます。保存された下書きは `Up` （上矢印キー）で呼び出せます。入力欄が空の場合、`Esc` を2回押すとリワインド（巻き戻し）メニューが開き、過去の時点のコードや会話を復元したり要約したりできます。)

Claude は編集前にファイルのスナップショットを取るので、編集結果ごと前の状態に戻せます。steer や `Esc` では取り返せないところまで進んだときの最終手段です。

> [!tip]
> 中断系は地道に修正が入っています（`v2.1.160`：ターン開始直後の `Esc` 取りこぼし、`v2.1.141`：vim モード中の `Ctrl+C`）。効かないと感じたら、まずバージョンを上げてみてください。

## Ctrl+S：書きかけのプロンプトを棚上げする（stash）

キューとは逆向きで、**いま入力している途中のプロンプトを失わずに、先に別の用事を片付ける**操作です。

1. 書いている途中で `Ctrl+S` を押すと、下書きが退避されて入力欄が空になる（ステータスに「Stashed (auto-restores after submit)」と表示）
2. 別の質問やコマンドを入力して送信する
3. その送信が終わると、退避した下書きが入力欄に自動で戻る

イメージは「`git stash` のプロンプト版」です。長い指示を組み立てている最中に急ぎの短い確認を挟みたくなっても、書きかけをどこかに退避させる必要がありません。

公式のキーバインドは `chat:stash`（デフォルト `Ctrl+S`）。退避スロットは1つで、戻す前にもう一度 `Ctrl+S` すると上書き、不要なら `Ctrl+C` で破棄できます。キーバインドは `/keybindings`（`~/.claude/keybindings.json`）で変更でき、カスタマイズ対応は `v2.1.18` 以降です。単一スロットを履歴と統合する要望（[#20806](https://github.com/anthropics/claude-code/issues/20806)）も出ています。

> [!note]
> `Ctrl+S` は文脈で役割が変わります。通常の入力欄では上記のスタッシュ、`Ctrl+R`（履歴検索）の最中は検索スコープの切り替えです（`historySearch:cycleScope`：このセッション → このプロジェクト → 全プロジェクト）。

## Shift+Tab：権限モードと plan

`Shift+Tab` でパーミッションモードを循環します。

- **default**：ファイル編集・コマンド実行のたびに確認
- **acceptEdits**：編集や `mkdir`・`mv` などを確認なしで実行
- **plan**：ソースを編集せず計画だけ立てる（`Shift+Tab` を2回で入る）
- **auto**：すべての操作を背後の安全チェックで評価（research preview）

「まず計画だけ立てさせて、レビューしてから実装」という二段構えは plan モードで作れます。作業の自由度をその場で締めたり緩めたりできる、進行制御の一部です。

## Ctrl+B / Ctrl+X Ctrl+K：バックグラウンドと一括停止

- `Ctrl+B`：実行中の bash コマンドやエージェントをバックグラウンドへ退避（tmux ユーザーは2回押し）。長いビルドやテストを走らせたまま、別の作業を続けられます。
- `Ctrl+X` `Ctrl+K`：このセッションで動いているバックグラウンドサブエージェントを全停止（3秒以内に2回で確定）。

## Ctrl+T：タスクリスト

`Ctrl+T` でタスクリストの表示を切り替えます。複数ステップの作業で、何が pending／in progress／完了かを把握しながら進められます。

## /btw：止めずに脇道の質問をする

`/btw` は、作業を止めずに脇で質問できるコマンドです。Claude が処理している最中でも実行でき、本流のターンを中断しません。ツールアクセスはなく（いまコンテキストにある情報だけで答える）、やり取りは会話履歴にも残りません。「さっき読んだ設定ファイルの名前なんだっけ」を、本流を乱さず確認したいときに向いています。

## まとめ

作業中の介入は、目的で選ぶと迷いません。詳細は冒頭の早見表に集約してあります。

- **方向を直す・指示を足す** → 止めずに `Enter`（steer）
- **いま動いている処理を止めて指示し直す** → `Esc`（作業は保持）
- **編集結果ごと過去に戻す** → `Esc` `Esc`（空入力で rewind）
- **書きかけを失わず別の指示を先に送る** → `Ctrl+S`（stash、送信後に自動復帰）
- **権限・計画を切り替える** → `Shift+Tab`（plan は2回）

Claude Code は「走らせながら舵を切る」のは得意な一方、「複数の指示を積んで後でまとめて流す」「queue の途中の1件だけ取り消す」はまだ弱く、要望が継続中です。この非対称を踏まえて `Enter` と `Esc` を使い分けるだけで、無駄な全体中断は大きく減らせます。

## 出典

- [Claude Code Docs — Interactive mode](https://code.claude.com/docs/en/interactive-mode)
- [Claude Code Docs — How Claude Code works](https://code.claude.com/docs/en/how-claude-code-works)
- [Claude Code Docs — Customize keyboard shortcuts（keybindings）](https://code.claude.com/docs/en/keybindings)
- [Claude Code Docs — Changelog](https://code.claude.com/docs/en/changelog)
- anthropics/claude-code Issues: [#34835](https://github.com/anthropics/claude-code/issues/34835) / [#50246](https://github.com/anthropics/claude-code/issues/50246) / [#20806](https://github.com/anthropics/claude-code/issues/20806)（stash 改善要望） / [#48802](https://github.com/anthropics/claude-code/issues/48802) / [#6147](https://github.com/anthropics/claude-code/issues/6147)
