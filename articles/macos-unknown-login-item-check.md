---
title: "Macのログイン項目に見慣れない項目が出た時の調べ方（sfltool dumpbtm）"
emoji: "🐸"
type: "tech"
topics: ["mac", "macos", "security", "cli"]
published: true
---

macOS の「ログイン項目と機能拡張」に見慣れない項目が出てきた時に、その正体を特定する手順を解説します。名前で検索しても分からないものを、実体の実行ファイルまで辿って「消していいのか、残すべきなのか」を自分で判断できるようにします。

きっかけは、システム設定 > 一般 > ログイン項目と機能拡張に、インストールした覚えのない項目が追加されていたことでした。「StaffPad Ltd って何？見たこともない名前だし怪し過ぎる…。」

![システム設定のログイン項目と機能拡張に StaffPad Ltd が表示されている](/images/macos-unknown-login-item-check/login-items.png)

## TL;DR

追加のツールは不要で、macOS 標準のコマンドだけで実体まで辿れます。
今回は 2 で決着したので 3 まではやってません。

1. `sfltool dumpbtm` で実行ファイルのパスを特定する
2. `find` でファイルの並びを見て、正規インストールの構成かどうか確認する
3. それでも心当たりが出なければ `codesign` で署名の開発元を照合する

## Step 1 sfltool dumpbtm で実体を出す

`sfltool dumpbtm` は、macOS が管理しているバックグラウンド項目（Background Task Management）の登録内容をダンプするコマンドです。設定画面に出ていた名前で絞り込みます。

```bash
sfltool dumpbtm | grep -i -A 15 -B 5 "StaffPad"
```

実行すると管理者認証（パスワードまたは Touch ID）を求められます。これは正常な挙動です。表示するだけのコマンドで何も削除しません。
出力から、該当部分だけを抜き出します。

```text
 #21:
                 UUID: 5DE2BC10-F63F-41F1-A4DB-4AF1506ED6E8
                 Name: StaffPad Ltd
       Developer Name: StaffPad Ltd
                 Type: developer (0x20)
          Disposition: [disabled, allowed, visible, not notified] (0x2)
           Identifier: StaffPad Ltd
  Embedded Item Identifiers:
    #1: 16.com.muse.authservice

 #22:
                 UUID: BE38FF3E-994C-457B-8A21-D647E2A177A3
                 Name: com.muse.authservice
       Developer Name: StaffPad Ltd
      Team Identifier: 6ZE93BN9HZ
                 Type: legacy daemon (0x10010)
          Disposition: [enabled, disallowed, visible, notified] (0x9)
           Identifier: 16.com.muse.authservice
                  URL: file:///Library/LaunchDaemons/com.muse.authservice.plist
      Executable Path: /Library/PrivilegedHelperTools/com.muse.authservice
    Parent Identifier: StaffPad Ltd
```

`#21` が設定画面に表示されていた項目そのもの、`#22` がその中身です。`Executable Path` に実行ファイルの絶対パスが出ています。ここで「StaffPad Ltd の正体は `com.muse.authservice` である」ということが判明しました。

## 出力の読み方

設定画面に出る名前は、アプリ名ではなく**コード署名の開発元名**です。インストールしたアプリの名前と一致しないことは珍しくありません。表示名で検索しても実体に届かないのはこのためで、`Executable Path` まで辿るのが早道です。主なフィールドは次のように読みます。

| フィールド                                             | 読み方                                                                                           |
| ------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| `Developer Name`                                  | コード署名の開発元。設定画面に出ている名前はこれ                                                                      |
| `Type`                                            | `developer` は項目を束ねる箱。`legacy daemon` や `app` が実体                                              |
| `Executable Path`                                 | 実行ファイルの絶対パス。ここが目的地                                                                            |
| `URL`                                             | 登録元の plist。`/Library/LaunchDaemons/` にあるものは全ユーザーに影響する                                         |
| `Embedded Item Identifiers` / `Parent Identifier` | 箱と中身の対応関係。どの項目がどれを抱えているか辿れる                                                                   |
| `Disposition`                                     | `enabled` / `disabled` はその項目が実行対象として有効かどうか、`allowed` / `disallowed` は設定画面でユーザーがオン・オフどちらにしているか |
| `Team Identifier`                                 | Apple が発行した開発者 ID。Step 3 の照合に使う                                                               |

## Step 2  ファイル側から裏取りする

パスが分かったら、ファイルシステム側にどう置かれているかを見ます。

```bash
find /Library ~/Library -iname "*muse*" -o -iname "*staffpad*" 2>/dev/null
```

無関係な一致（Adobe のレンズプロファイルやブラウザ拡張のアイコンなど）もかなり混ざるので、関係のある行だけ拾います。

```text
/Library/Application Support/Muse Hub
/Library/PrivilegedHelperTools/com.muse.authservice
/Library/LaunchDaemons/com.muse.authservice.plist
/Library/Logs/com.muse.authservice
~/Library/Application Support/Muse Hub
~/Library/Application Support/MuseSampler
```

`Application Support` にアプリ本体のデータ、`LaunchDaemons` に登録用の plist、`PrivilegedHelperTools` に権限付きヘルパー。この 3 つが揃っているのは、権限付きヘルパーを持つアプリの正規インストール構成そのものです。ここで Muse Hub というインストール元まで確定します。

ログの日付も手掛かりになります。

```text
/Library/Logs/DiagnosticReports/ExcResource_com.muse.authservice-2026-08-13-092210.diag
```

前日にこのプロセスが診断レポートを出していました。自分がアプリを起動・更新した日と一致すれば、「その操作の結果として増えた」という筋がさらに固まります。

## Step 3  署名を照合する

Step 2 まで進んでも心当たりのあるアプリが 1 つも出てこない場合は、署名を見ます。今回はここに来る前に決着したので、以下は手順の紹介です。

```bash
codesign -dv --verbose=4 /Library/PrivilegedHelperTools/com.muse.authservice
spctl -a -vv -t install /Applications/対象.app
```

`codesign` の出力で見るのは `Authority=Developer ID Application: <会社名> (<Team ID>)` と `TeamIdentifier` の行です。Step 1 で得た `Team Identifier` と一致するかを照合します。署名そのものが無い、あるいは `Authority` の会社名が名乗りと食い違う場合は、そこで初めて疑う段階に入ります。

## 出力をそのまま AI に貼り聞くと良い

`sfltool dumpbtm` の出力を AI に貼った上で「`Embedded Item Identifiers` とこのファイルパスの対応を説明して」と指示しフィールド同士の関係を説明させると楽です。

注意としては、AI が削除コマンドを提案してきても、そのまま実行しないでください。パスは必ず自分の手元の出力と突き合わせます。ログイン項目まわりには、`sfltool resetbtm` のようにサードパーティの項目を一括でリセットしてしまう破壊的なコマンドも存在します。

## 戻すのか、消すのか

ここまでで判断材料は揃っているはずです。

そのアプリを使っているなら、オンに戻して問題ありません。オンにする＝常時 CPU を消費し続ける、という意味ではなく、macOS がバックグラウンドでの実行を許可しているという設定です。権限付きヘルパーはアップデートやライブラリのインストールなど、管理者権限が必要な処理で使われるので、オフのままだとアプリ側の機能が正常に動かないことがあります。

使っていないなら、その項目だけを消すのではなく、アプリ本体を正規の手順でアンインストールします。項目だけ消しても、`LaunchDaemons` や `PrivilegedHelperTools` のファイルは残骸として残ります。

## どうしても手動で消す場合

公式のアンインストーラを先に探してください。手動削除は最後の手段です。

:::message alert
以下は `/Library/LaunchDaemons/` 配下を直接操作する手順です。他のアプリの動作にも関わる領域なので、パスを 1 文字も違えずに実行してください。作業前に Time Machine のバックアップがあることを確認することを勧めます。
:::

順序は「停止 → plist 削除 → バイナリ削除」です。逆にやると、ファイルを消したそばから再登録されることがあります。

```bash
sudo launchctl bootout system/com.muse.authservice
```

停止できたら、Step 1 の `URL` と `Executable Path` に出ていたパスを削除します。パスは記事の例をコピーせず、自分の出力から取ってください。

## 答え合わせ

今回の正体は、`/Applications/Audacity 4.app` が同梱でインストールした Muse Hub の認証用ヘルパー `com.muse.authservice` でした。Audacity の開発元である Muse Group は MuseScore や StaffPad を傘下に持っていて、このヘルパーの署名開発元名が StaffPad Ltd になっているため、設定画面には楽譜アプリの会社名だけが見えていた、という構図です。

`find` で見つかった Muse Hub は中間コンポーネントで、`/Applications` を確認するまで最終的な入口には辿り着けませんでした。`stat -f "%SB %N" /Library/LaunchDaemons/com.muse.authservice.plist` で登録日時を見ると 2 月で、直近にインストールしたツールとは無関係だという切り分けもできます。

見慣れない項目に出会ったら、名前で検索する前に `sfltool dumpbtm` でパスまで辿る。それだけで、消すか残すかを自分で決められるようになります。