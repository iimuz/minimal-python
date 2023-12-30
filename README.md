---
title: Minimal Python Settings
date: 2022-06-08
lastmod: 2023-12-31
---

## minimal python

pythonのコードを書くときに利用する最小限の設定です。
ただし、ここでの対象は実行用スクリプトを開発することを想定しています。
そのため、下記は対象外です。

- pipインストール可能なライブラリ開発
- pipインストール可能なツール開発

## ファイル構成

- フォルダ
  - `.devcontainer`: VSCode Remote Containersの設定を記述します。
  - `.github`: GitHubのworkflow設定を記述します。
  - `.vscode`: VSCodeの基本設定を記述します。
  - `src`: 開発するスクリプトを格納します。
- ファイル
  - `.cspell.json`: [CSpell](https://cspell.org/)の設定を記述します。
  - `.editorconfig`: Editorの共通設定を記述します。
  - `.gitignore`: 以下のignore設定を結合しています。
    - [python gitignore](https://github.com/github/gitignore/blob/main/Python.gitignore)
    - [node gitignore](https://github.com/github/gitignore/blob/main/Node.gitignore)
  - `.prettierignore`: [prettier](https://prettier.io/)の設定を記述します。
  - `.sample.env`: 環境変数のサンプルを記載します。利用時は`.env`に変更して利用します。
  - `docker-compose.yml`, `Dockerfile`: docker環境の設定を記述します。
  - `dprint.json`: dprintの設定を記述します。
  - `LICENSE`: ライセンスを記載します。MITライセンスを設定しています。
  - `package.json`: nodeのパッケージ情報を記載します。
  - `pyproject.toml`/`setup.py`/`setup.cfg`: python バージョンなどを明記します。
  - `README.md`: 本ドキュメントです。
  - `Taskfile.yml`: [task](https://taskfile.dev/)コマンドの設定を記述します。

## dockerを用いた実行方法

コマンドのみを実行できれば良い場合は下記のように実行します。

```sh
docker compose run --rm -it app
```

vscode と同様の開発環境で起動する場合は下記のように実行します。

```sh
docker compose -f docker-compose.yml -f .devcontainer/docker-compose.extend.yml -f .devcontainer/docker-compose.local.yml run --rm -it app
```

## ローカル環境の構築

事前に下記が利用できるように環境を設定してください。

- [node.js](https://nodejs.org/en): 開発環境のlinterに利用します。
- [python](https://nodejs.org/en)
- [task](https://taskfile.dev/): タスクランナーとして利用します。

仮想環境などの構築は下記のコマンドで実行します。

```sh
# 実行だけできればよい場合
task init
# 開発環境もインストールする場合
task init-dev
```

## Taskfile

実行可能なタスク一覧は下記のコマンドで確認してください。

```sh
task -l
```

## code style

コードの整形などはは下記を利用しています。

- json, markdown, toml
  - [dprint](https://github.com/dprint/dprint): formatter
- python
  - [ruff](https://github.com/astral-sh/ruff): python linter and formatter.
  - [mypy](https://github.com/python/mypy): static typing.
  - docstring: [numpy 形式](https://numpydoc.readthedocs.io/en/latest/format.html)を想定しています。
    - vscodeの場合は[autodocstring](https://marketplace.visualstudio.com/items?itemName=njpwerner.autodocstring)拡張機能によりひな型を自動生成できます。
- yml
  - [prettier](https://prettier.io/): formatter

## VSCode remote containers

VSCode のリモートコンテナを利用した環境構築を利用できます。
下記拡張機能をインストールし、 `Dev Containers: Reopen in Container` コマンドを実行することで Dockerfile の環境で VSCode を利用することができます。

- [Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)

Container 環境にユーザを作成するため環境変数に下記の設定を追加してください。環境変数がない場合は、ユーザ名`vscode`、`UID=1000`, `GID=1000`でユーザを作成します。

- `USER_UID`: コンテナ内で利用するユーザの ID
- `USER_GID`: コンテナ内で利用するユーザのグループ ID

リポジトリのコード実行に最低限必要な環境を `Dockerfile`, `docker-compose.yml` で定義しています。そのため、VSCode 用に必要な追加設定については、 `.devcontainer/docker-compose.extend.yml` に記載します。
また、共通設定ではなく個人設定として、docker 環境にマウントするディレクトリを増やす場合などは、 `.devcontainer/docker-compose.local.yml` を編集します。
編集前に git で変更を検知しないように下記の設定を行ってください。下記の設定を行うことで、変更をリポジトリにコミットしないで修正可能になります。

```sh
git update-index --skip-worktree .devcontainer/docker-compose.local.yml
```

例えば、 `git worktree` などを利用している場合は、本リポジトリのみをマウントしても git が有効にならないため、 `docker-compose.local.yml` に下記のような定義を追加して対応できます。

```yml
services:
  app:
    volumes:
      - type: bind
        source: $HOME/path/to/git/worktree/root
        target: $HOME/path/to/git/worktree/root
      - type: bind
        source: $PWD
        target: $PWD
```

python 仮想環境を named volume で用意しています。WSL で実行する場合に、named volume をマウントした位置のホスト側に root 権限で空のフォルダが作成されます。そのため、先に `mkdir .venv` などでフォルダを作成しておくことで root 権限でのフォルダ生成を回避することができます。
