# pr-based-dev-guide
PRベースの開発をすることで以下を目指すためのガイド。`gh` ([GitHub CLI](https://cli.github.com/))の利用を想定する。

- [自動生成リリースノート](https://docs.github.com/ja/repositories/releasing-projects-on-github/automatically-generated-release-notes)
  - 本リポジトリではmainブランチへのPRがマージされた時にリリースが作成され、PRとそのPRに関連したIssueの内容からリリースノートが自動生成される。

本ガイドの手順を試す場合は本リポジトリをフォークしてクローン。
```sh
gh repo folk terabayashik/pr-based-dev-guide --clone
```

## ブランチ戦略
ここでは以下のブランチ戦略を想定している。
- **main**: メインブランチ。このブランチへのコミットはしない。**develop**ブランチからのPRのみをマージする。プロダクトとしてリリースするもの。リリースノートの自動生成はこのブランチへのマージをトリガーとしている。
- **develop**: 開発中ブランチ。このブランチへのコミットは**しない**。各Issue専用のブランチからのPRのみをマージする。
- **Issue専用ブランチ**: Issueに関連付けられたブランチ。`gh`コマンドで発行する場合のブランチ名は`1-i-found-a-bug`のようにIssue IDとタイトルから構成される。

## Issueを作成する
```sh
gh issue create --title "I found a bug" --body "Nothing works." --label bug
```
Issue作成時に指定するラベルに応じて、リリースノート生成時のカテゴリが分類される。
### 主要なオプション
- `-t`, `--title`: タイトル。指定しない場合はプロンプトが表示される。
- `-b`, `--body`: 本文。指定しない場合はプロンプトが表示される。
- `-l`, `--label`: ラベル。デフォルトでサポートされるタグは以下。
  - bug
  - documentation
  - duplicate
  - enhancement
  - good first issue
  - help wanted
  - invaild
  - question
  - wontfix
- `-e`, `--editor`: プロンプトをスキップしてエディタを使用する。

`-t`, `-b`, `-e`などを指定することでプロンプトがスキップされる場合はラベルを指定するプロンプトが表示されない。これらを指定する場合は`-l`を忘れずに指定すること。

## Issueに取り組む
```sh
# 目的のIssueを探す
gh issue list
# 目的のIssueのためのブランチを作成してチェックアウト
gh issue develop <number> --base develop --checkout
```
### 主要なオプション
- `-b`, `--base`: 派生元のブランチ名
- `-c`, `--checkout`: ブランチ作成後にチェックアウトを行う

ここでは`package.json`のバージョンを修正することで擬似的にIssueの解決を行なったものとする。
```sh
jq '.version = "1.0.0"' package.json > temp.json && mv temp.json package.json
git add ./package.json
git commit -m "Fix bug"
git push
```

## PRを作成する
Issue用のブランチ(例: `1-i-found-a-bug`)
```sh
gh pr create --title "The bug is fixed" --body "Everything works again" --label bug --base develop
```
### 主要なオプション
- `-t`, `--title`: タイトル。
- `-b`, `--body`: 本文。
- `-B`, `--base`: マージ先のブランチ。
- `-l`, `--label`: ラベル。デフォルトでサポートされるタグは以下。
  - bug
  - documentation
  - duplicate
  - enhancement
  - good first issue
  - help wanted
  - invaild
  - question
  - wontfix
- `-e`, `--editor`: プロンプトをスキップしてエディタを使用する。

## PRをマージする
```sh
gh pr merge <number>
```
これで、**develop**ブランチに**1-i-found-a-bug**ブランチの変更がマージされた。この時点でローカルでは**develop**ブランチにいることを確認する。

続いて、**develop**ブランチの変更を**main**ブランチにマージする。
```sh
gh pr create
gh pr merge
```

以上で変更が**main**ブランチに反映され、GitHub Actionsによりリリースが作成される。
