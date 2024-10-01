---
title: "GitHub で Organization のリポジトリを個人アカウントにフォークする方法"
emoji: "🐸"
type: "tech"
topics: ["git","github"]
published: true
---

GitHub の UI では、自分が管理している Organization のリポジトリを自分の個人アカウントにフォークすることができません。しかし、少しの手順を踏めば同等のことが可能です。

その方法を説明し、続けて個人リポジトリでの作業内容をフォーク元である Organization に反映させる手順を解説します。

## Organization のリポジトリを個人アカウントにフォークする手順

### 1. Organization のリポジトリをローカルにクローン

フォークしたい Organization のリポジトリを、ローカルにクローンします。

```bash
git clone https://github.com/organization_name/repository_name.git
```

### 2. 個人アカウントに新しいリポジトリを作成

個人アカウントで、新しいリポジトリを作成します。  
リポジトリ名は元のものと異なっていてもいけますが、揃えた方が良いでしょう。

### 3. リモートURLを変更

クローンしたリポジトリのリモートURLを、個人アカウントの新しいリポジトリに向けるよう設定します。

```bash
cd repository_name
git remote set-url origin https://github.com/your_personal_account/new_repository_name.git
```

### 4. リポジトリをプッシュ

ローカルで作業した内容を、自分の個人アカウントの Github リポジトリにプッシュします。

```bash
git push -u origin main
```

これで、Organization リポジトリの内容を個人アカウントで管理できるようになりました。
今後の更新作業は、個人アカウントのローカルレポジトリで行います。

---

## 個人リポジトリでの変更を Organization に反映させる方法

個人アカウントで行った変更を Organization リポジトリに反映するために、まずはプルリクエストをします。そして、Organization のリポジトリ側(個人開発なら自分、チーム開発ならメンバー)で確認・承認をすることで反映されます。

### 1. Organization のリポジトリをリモートに追加

まず、個人リポジトリに Organization のリポジトリを `upstream` という名前 (upstream 出なくても、何でもOK) で追加します。これにより、個人リポジトリと Organization のリポジトリとの同期が取れるようになります。

```bash
git remote add upstream https://github.com/organization_name/repository_name.git
```

### 2. 最新の状態を取得

複数人での開発の場合は、リモートの Organizasion リポジトリが最新の状態かどうかを確認します。更新があった場合は自分のローカルリポジトリをアップデートします。これを行わないとコンフリクトが発生する可能性があります。

```bash
git fetch upstream
git merge upstream/main
```

### 3. 個人リポジトリに変更をプッシュ

個人アカウントのローカルリポジトリで作業を進め、変更が完了したら、その内容を個人アカウントのリモートリポジトリにプッシュします。

```bash
git add .
git commit -m "Your commit message"
git push origin main
```

### 4. プルリクエストを送信

最後に、GitHub の個人アカウントのリポジトリからプルリクエストを作成します。プルリクエストのターゲットは、元の Organization リポジトリの`main`ブランチで、比較対象として自分の`main`ブランチを設定します。

### 完了

これで、個人アカウントのリポジトリと Organization のリポジトリを簡単に同期できるようになりました！
