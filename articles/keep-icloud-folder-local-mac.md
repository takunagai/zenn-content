---
title: "Mac: iCloud Drive の指定フォルダをローカルに保持し続ける方法"
emoji: "🐸"
type: "tech"
topics: [mac, 開発環境, markdown, iawriter, obsidian]
published: true
---

## はじめに

MacOS の iCloud Drive には、ストレージ容量が不足してきた時に自動でフォルダやファイルをクラウドへ移動する「最適化されたストレージ」機能があります。通常は便利な機能ですが、ローカルにファイルの実体がなくなるため、不便が生じることがあります。

私は iA Writer や Obsidian などの書類を管理するアプリを iCloud で同期させているのですが、以下のような不便が発生しました。

- オフラインでの作業に支障が出る
- アプリがそれらのファイルを参照するときにタイムラグが生じる
- Gitバージョン管理に問題が生じる

これらの問題を解決するため、特定のフォルダを**常にローカルに保持**するようにしました。この記事では、その方法を紹介します。

## 解決方法

特定のファイルを常にローカルに保持するために、**定期的に対象の全ファイルのアクセス時間(atime)を更新**します。

## 実装手順

今回は、iA Writer の保存フォルダを常にローカルに保持するように設定します。他のアプリケーションの保存フォルダを保持する場合は、パスを変更してください。

### 1. atime 更新の有効性確認

まず、システムで `atime`（アクセス時間）の更新が有効になっているか確認します。

```sh
mount | grep iCloud
```

:::message alert
もし出力に `noatime` が含まれている場合は、後述の「トラブルシューティング」セクションを参照してください。
:::

### 2. launchd でスケジュール設定

macOS では cron よりも launchd（LaunchAgents）が推奨されています。ログインユーザー環境やスリープ復帰との相性が良く、iCloud 配下の定期処理に適しています。

#### 2-1. ユーザー名を確認

以下のコマンドで自分のユーザー名を確認します。この後の設定で使用します。

```sh
whoami
```

#### 2-2. コマンドの動作確認

設定前に、ターミナルでコマンドが正常に動作するか確認します。`<USERNAME>` は先ほど確認したユーザー名に置き換えてください。

```sh
/usr/bin/find "/Users/<USERNAME>/Library/Mobile Documents/27N4MQEA55~pro~writer/Documents" -type f -exec /usr/bin/touch -a {} \;
echo $?
```

`0` と表示されれば成功です（ファイル数が多いと少し時間がかかります）。

#### 2-3. plist ファイルを作成

LaunchAgents 用のディレクトリを作成し、設定ファイルを作成します。

```sh
mkdir -p ~/Library/LaunchAgents
nano ~/Library/LaunchAgents/com.user.touch-iawriter.plist
```

以下の内容を貼り付けます。`<USERNAME>` は実際のユーザー名に置き換えてください。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
 "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>com.user.touch-iawriter</string>

    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/find</string>
        <string>/Users/<USERNAME>/Library/Mobile Documents/27N4MQEA55~pro~writer/Documents</string>
        <string>-type</string>
        <string>f</string>
        <string>-exec</string>
        <string>/usr/bin/touch</string>
        <string>-a</string>
        <string>{}</string>
        <string>;</string>
    </array>

    <!-- 12時間 = 43200秒 -->
    <key>StartInterval</key>
    <integer>43200</integer>

    <!-- ログ出力 -->
    <key>StandardOutPath</key>
    <string>/Users/<USERNAME>/cron_log.txt</string>
    <key>StandardErrorPath</key>
    <string>/Users/<USERNAME>/cron_log.txt</string>

    <!-- ログイン時に1回実行 -->
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

nano の操作: `Ctrl+O` → Enter で保存、`Ctrl+X` で終了。

:::message
**重要**: plist 内では `~` は展開されません。必ず `/Users/<USERNAME>/...` のように絶対パスで記述してください。
:::

#### 2-4. 構文チェックとジョブの有効化

```sh
# 構文チェック（"OK" と表示されれば問題なし）
plutil ~/Library/LaunchAgents/com.user.touch-iawriter.plist

# ジョブを有効化
launchctl load ~/Library/LaunchAgents/com.user.touch-iawriter.plist
```

#### 2-5. 動作確認

```sh
# 登録されているか確認
launchctl list | grep touch-iawriter

# ログを確認
tail -n 20 ~/cron_log.txt
```

:::message
**再起動後も維持されます**: plist が `~/Library/LaunchAgents/` にある限り、ログイン時に自動でロードされます。`RunAtLoad=true` により、ログイン直後に1回実行され、その後12時間ごとに実行されます。
:::

## トラブルシューティング

### launchd が動作しない場合

よくある原因:

- **plist 内で `~` を使っている**: plist 内では `~` は展開されません。絶対パス `/Users/<USERNAME>/...` を使用してください
- **対象ディレクトリが存在しない**: iCloud が未同期のタイミングで実行されると失敗します
- **構文エラー**: `plutil` コマンドで構文チェックを実行してください

### atime 更新が無効の場合の代替方法

`atime` の更新が無効（`noatime`）の場合は、`cat` コマンドを使用する方法が有効です。

```sh
find $HOME/Library/Mobile\ Documents/com~apple~CloudDocs/iAWriter -type f -exec cat {} > /dev/null \;
```

### launchd ジョブの停止・削除方法

設定を停止または削除したい場合:

```sh
# 停止（plist は残る）
launchctl unload ~/Library/LaunchAgents/com.user.touch-iawriter.plist

# 完全に削除
launchctl unload ~/Library/LaunchAgents/com.user.touch-iawriter.plist
rm ~/Library/LaunchAgents/com.user.touch-iawriter.plist
```

### cron から移行する場合

以前 cron を使用していた場合は、二重実行を防ぐため cron 設定を削除してください。

```sh
# 現在の cron 設定を確認
crontab -l

# cron 設定を編集して該当行を削除
crontab -e

# または、cron をすべて削除（他で使用していない場合）
crontab -r
```

## よくある質問

### Q: フォルダの「ダウンロードを保持」機能では不十分？

A: 「ダウンロードを保持」機能は基本的に有効ですが、以下の理由で不安定な場合があります。

- ストレージ容量が極端に少なくなった場合の挙動が不確実
- システムアップデートで動作が変更される可能性

### Q: 「最適化されたストレージ」を無効にする方法は？

A: システム設定 > Apple ID > iCloud で「Macのストレージを最適化」を無効にできますが、すべての iCloud Drive ファイルがローカルに保存されるため、ストレージ効率が悪くなります。

## まとめ

この方法を使うことで、以下のようなメリットがあります。

- 特定のフォルダを確実にローカルに保持
- クラウド同期の利点を維持
- ストレージ容量の自動最適化も活用

必要なフォルダだけを選択的にローカルに保持できるため、iCloud Drive の利点を活かしながら、作業効率を維持することができます。
