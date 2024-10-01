---
title: "Git で任意のブランチを新プロジェクトとして独立させる方法"
emoji: "🐸"
type: "tech"
topics: ["git","github"]
published: true
---

Git で既存プロジェクトにある任意のブランチを、履歴を保持させつつ別の新しいプロジェクトとして独立させる手順のメモです。
（履歴が不要なら、ローカルPCでプロジェクトフォルダを丸ごとコピーして、Git を初期化すれば OK。）

## 手順

今回は既存プロジェクトにある `new-project` というブランチを、新プロジェクトとして新しいリポジトリに移行することとします。

### 1. 新しいリポジトリを作成

GitHub で移行先となる新プロジェクト用のリポジトリを作成します。ここではリポジトリ名は `new-project-repo` とします。作成直後なので、この時点では空っぽの状態です。

### 2. 作業するブランチへ移動

既存プロジェクトでターミナルを開き、`new-project` ブランチに移動します。

```bash
git checkout new-project
```

### 3. 不要なコミットを整理

不要なコミットを削除したい場合、`git rebase` や `git filter-branch` などで履歴を整理します。

### 4. 新しいリモートリポジトリを追加

ローカルリポジトリに新しいリモートリポジトリを追加します。ここで使う「リモート名」は、リポジトリへのショートカットとして機能します。デフォルトで設定される `origin` が存在しますが、今回は新しいリポジトリ用に別のリモート名 `new-origin` を指定します (リモート名は自由に設定可)。

```bash
git remote add new-origin https://github.com/username/new-project-repo.git
```

現在のリポジトリに設定されているリモートリポジトリの一覧を確認しておきましょう。

```bash
git remote -v
```

↓ 応答

```bash
new-origin    https://github.com/username/new-project-repo.git (fetch)
new-origin    https://github.com/username/new-project-repo.git (push)
origin        https://github.com/username/old-project-repo.git (fetch)
origin        https://github.com/username/old-project-repo.git (push)
```

### 5. 新しいリポジトリにプッシュ

`new-project` ブランチを新しいリモートリポジトリの `main` ブランチとしてプッシュします。

```bash
git push new-origin new-project:main
```

このコマンドにより、`new-project` ブランチの内容が新しいリポジトリの `main` ブランチとして保存されます。無事移行できてるか確認しましょう。

### 6. コードエディタや IDE で新プロジェクトの設定

Visual Studio Code や Jetbrains IDE で、 新プロジェクトの環境を整えます。新プロジェクトを作成し、新しく作成したリポジトリをクローン。`npm install` で package.json に記載されたライブラリを取り込みます。ローカルサーバーを立ち上げ、ローカル環境での動作確認が問題ないことを確認します。

### 7. 元のプロジェクトでのブランチ整理

移行元のプロジェクトで、不要になった移行済みの `new-project` ブランチを削除します。

```bash
git branch -d new-project
```

### 完了 👍

以上の手順で、既存プロジェクトにある任意のブランチを、新しいプロジェクトとして切り離すことができました。
