---
title: "Mac ターミナルの設定ファイル .zprofile と .zshrc をやさしく解説"
emoji: "🐸"
type: "tech"
topics: [mac, terminal, shell]
published: true
---

ターミナル設定で必ず出てくるのが `.zprofile` と `.zshrc`。
「どちらに何を書くのか？」を整理しつつ、**VSCode や GUI アプリまで PATH/環境変数が正しく反映される**方法まで解説します。

## 結論（最初にざっくり）

- **.zprofile** … *ログインシェル* のとき一度だけ読む → **PATH/環境変数/初期化** を書く
- **.zshrc** …… *インタラクティブシェル* が起動するたび読む → **補完/エイリアス/プロンプト** を書く
- Terminal.app、**設定で "デフォルトのログインシェル"** となっていることを確認
- **VSCode などはデフォルトで非ログインシェル** → `.zprofile` を読ませるには設定変更が必要
- **恒久的に GUI アプリに PATH を通すには LaunchAgent を使う** (一時的なら `launchctl setenv`)

### まとめ早見表

- Homebrew 初期化 → **.zprofile**
- PATH 追加 → **.zprofile**
- 環境変数 → **.zprofile**
- 補完（`compinit` など）→ **.zshrc**
- エイリアス → **.zshrc**
- プロンプト（Starship）→ **.zshrc**
- Zellij/Tmux 自動起動 → **.zshrc**

---

## 読み込みの違い（まず覚えるべきポイント）

| ファイル        | いつ読む？                      | 代表的に書くもの                                    |
| ----------- | -------------------------- | ------------------------------------------- |
| `.zprofile` | ログインシェル起動時に **一度だけ**       | PATH、環境変数、Homebrew 初期化など“前提”設定              |
| `.zshrc`    | インタラクティブ zsh が起動する **たびに** | 補完（`compinit`）、エイリアス、プロンプト（Starship）など“操作系” |

> 注意: デフォルトの **VSCode などは非ログインシェル** → 設定変更しない限り `.zprofile` は読まれません。Terminal.app はログインシェル起動になっているはずだが、設定画面で確認すること。

---

## ベストプラクティス構成

1. **`.zprofile` に PATH/環境変数を定義**（重複を防ぐ）
2. **Terminal.app / VSCode 等を「ログインシェル起動」に変更**
3. **LaunchAgent で GUI にも環境を配布**

---

## 手順 1：`.zprofile` に環境変数を書く

```zsh:.zprofile
# Homebrew 初期化（Apple Silicon）
if [ -x /opt/homebrew/bin/brew ]; then
  eval "$(/opt/homebrew/bin/brew shellenv)"
fi

# 任意の環境変数
export ENABLE_BACKGROUND_TASKS=1

# PATH 追加（必要に応じて）
[ -d "$HOME/.local/bin" ] && export PATH="$HOME/.local/bin:$PATH"
[ -d "$HOME/.cargo/bin" ] && export PATH="$HOME/.cargo/bin:$PATH"
[ -d "$HOME/.composer/vendor/bin" ] && export PATH="$HOME/.composer/vendor/bin:$PATH"

# VSCode の code コマンド
[ -d "/Applications/Visual Studio Code.app/Contents/Resources/app/bin" ] \
  && export PATH="/Applications/Visual Studio Code.app/Contents/Resources/app/bin:$PATH"
```

> `.zshrc` に PATH を書くとタブを開くたびに重複するので、`.zprofile` にまとめるのがおすすめ。

---

## 手順 2：`.zshrc` に操作系の設定を書く

```zsh:.zshrc
# 補完
autoload -Uz compinit
compinit -C -i

# uv の補完
if (( $+commands[uv] )); then
  eval "$(uv generate-shell-completion zsh)"
fi

# eza（ls 代替）
if (( $+commands[eza] )); then
  alias ls='eza --icons --git'
  alias ll='eza -aahl --icons --git'
fi
alias l='clear && ls'

# Starship（プロンプト）
(( $+commands[starship] )) && eval "$(starship init zsh)"
```

---

## 手順 3：ログインシェルで起動する設定

これをやらないと `.zprofile` は読まれません。

### Terminal.app

- 環境設定 → 一般 → 開くシェルで "デフォルトのログインシェル" にチェックが入っていることを確認

### VSCode（統合ターミナル）

コマンドパレット(`Cmd+Shift+P`)を開き、"settings.json" を検索、"基本設定： 設定を開く（JSON）"を選択。
settings.json に以下を追記

```json:settings.json
"terminal.integrated.profiles.osx": {
  "zsh": { "path": "/bin/zsh", "args": ["-l", "-i"] }
},
"terminal.integrated.defaultProfile.osx": "zsh"
```

---

## 手順 4：GUI アプリにも PATH を伝える

ここでいう GUI アプリとは、Finder や Dock から直接起動するアプリのことです。例えば VSCode 本体、JetBrains 系 IDE（IntelliJ IDEA, PyCharm など）、Docker Desktop、GUI 版 Git クライアント（Sourcetree や Tower）、Xcode など。こうしたアプリは `.zprofile` や `.zshrc` を読まないため、PATH が通らない場合があります。

macOS の GUI は `launchd` が環境を配る仕組みです。`.zprofile` だけでは反映されません。

### 永続的に反映（推奨）

`~/Library/LaunchAgents/user.env.plist` を作成：

```xml:user.env.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key><string>user.env</string>
  <key>ProgramArguments</key>
  <array>
    <string>/bin/launchctl</string>
    <string>setenv</string>
    <string>PATH</string>
    <string>/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin</string>
    <string>/bin/launchctl</string>
    <string>setenv</string>
    <string>ENABLE_BACKGROUND_TASKS</string>
    <string>1</string>
  </array>
  <key>RunAtLoad</key><true/>
</dict>
</plist>
```

設定したら反映 (ログアウト→ログインで確実に反映)：

```bash
launchctl load ~/Library/LaunchAgents/user.env.plist
```

### 補足: 一時的に反映（ログアウトまで有効）

```bash
launchctl setenv PATH "/opt/homebrew/bin:/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin"
launchctl setenv ENABLE_BACKGROUND_TASKS "1"
```

---

## おわりに

`.zprofile` と `.zshrc` の住み分けを理解し、ログインシェル起動と GUI への反映をセットで押さえると、環境設定は安定します。
**「前提は .zprofile、操作は .zshrc、各ターミナルをログインシェルに設定、GUI は launchctl(LaunchAgents で永続化)」**――この並びを覚えておけば迷いません。

> 「道具は正しい場所に。場所が整えば、作業は驚くほど速くなる。」
