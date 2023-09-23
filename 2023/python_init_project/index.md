---
title: 'Pythonプロジェクトの作成（pyenv + Poetry）'
date: '2023-08-07T15:45:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Python
tags:
  - Python
  - Poetry
  - pyenv
---
# Pythonプロジェクトの作成（Python 3.11, pyenv + Poetry, VSCode）

## 記事作成時点のバージョン情報

- [pyenv 2.3.23](https://github.com/pyenv/pyenv)
- [Poetry 1.5.1](https://python-poetry.org/docs/#installation)
- [Python 3.11.4](https://www.python.org/downloads/)

## 定義・ディレクトリ構成

説明のため、プロジェクトディレクトリ名`my_project`、パッケージ名`my-project`、主要なモジュール名`my_project`とします。

以下のようなディレクトリ構成にすることを想定しています。

```plain
- my_project/
  - pyproject.toml
  - Dockerfile
  - my_project/
    - __init__.py
    - my_module.py
  - scripts/
    - __init__.py
    - main.py
  - tests/
    - __init__.py
    - test_my_module.py
```

## Python/Poetryのインストール

pyenvでPythonをインストールします。
記事作成時点で最新のリビジョン（`0.0.x`）を記載していますが、適宜新しいバージョンが出ているか確認し、
更新してください。
マイナーバージョン（`0.x.0`）を変更する場合、依存する予定のライブラリが動作するかなど、プロジェクトの要件と相談してください。

```shell
env PYTHON_CONFIGURE_OPTS="--enable-shared" pyenv install 3.11.4
```

`PYTHON_CONFIGURE_OPTS="--enable-shared"`は、PyInstallerが動作するようにするために設定しています。

- [pyenv and PyInstaller — PyInstaller 5.13.0 documentation](https://pyinstaller.org/en/stable/development/venv.html)

Poetryをインストールします。

- [Poetry Installation](https://python-poetry.org/docs/#installation)

```shell
# Linux, macOS, WSL
curl -sSL https://install.python-poetry.org | python3 -

# Windows (PowerShell)
(Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | python -
```

## Poetryプロジェクトの作成

Poetryのグローバル設定を変更し、Python仮想環境が`プロジェクトのディレクトリ/.venv`に作成されるようにします。
これは、VSCode拡張機能の[Pylance](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance)がPython仮想環境を認識できるようにする、または手動で設定しやすくするための変更です。

```shell
poetry config virtualenvs.in-project true
```

プロジェクトディレクトリ`my_project`を作成し、作業ディレクトリにします。

```shell
mkdir my_project
cd my_project
```

pyenvのPythonバージョン指定ファイル`.python-version`を作成します。

```shell
pyenv local 3.11.4
```

現在のディレクトリにPoetryプロジェクトを作成します。
対話形式でプロジェクトの設定（`pyproject.toml`の作成）をします。

```shell
poetry init
```

パッケージ名には、以下の文字が使用できます。

- ラテン文字（A-Z、大文字・小文字は区別されない）
- アラビア数字（0-9）
- ピリオド、アンダースコア、ハイフン（これらは区別されない、先頭または末尾に使用できない）

パッケージ名の仕様について、記事作成時点では以下が参考になります。

- [パッケージ名に関するPyPA仕様](https://packaging.python.org/en/latest/specifications/core-metadata/#name)
- [パッケージ名の正規化に関するPyPA仕様](https://packaging.python.org/en/latest/specifications/name-normalization/#name-normalization)
- [PEP 566 – Metadata for Python Software Packages 2.1](https://peps.python.org/pep-0566/)
- [PEP 508 – Dependency specification for Python Software Packages](https://peps.python.org/pep-0508/)

（おすすめ）依存ライブラリを聞かれますが、個人的には、操作がややこしく間違ったライブラリをインストールしてしまうのが怖いので、いったんスキップして後で設定することが多いです。

GitHubの`.gitignore`テンプレートをプロジェクトディレクトリにコピーします。

- [GitHubの`.gitignore`テンプレート](https://github.com/github/gitignore/blob/main/Python.gitignore)（[記事作成時点のバージョン](https://github.com/github/gitignore/blob/4488915eec0b3a45b5c63ead28f286819c0917de/Python.gitignore)）

（おすすめ）`.gitignore`を編集し、pyenvのPythonバージョン指定ファイル`.python-version`をGit管理から除外します。
`.python-version`はPythonバージョンをリビジョンまで固定するため、以下のようなケースで
完全に一致したバージョンのPythonをそれぞれインストールすることになり、不便になります。

- 複数の開発者がいる
- 複数の開発環境がある（コンピュータ、OS、仮想環境）
- 複数のPythonプロジェクトがある

```gitignore
# pyenv
#   For a library or package, you might want to ignore these files since the code is
#   intended to run in multiple environments; otherwise, check them in:
.python-version
```

主要なドキュメント`README.md`、主要なモジュールのディレクトリ`my_project/`、ファイル`my_project/__init__.py`を作成します。
これらのファイルは、プロジェクトに対してPoetryを動作させるために必要です（Poetry実行時にファイルが存在しない旨のエラーが出ます）。

```shell
echo "# my_project" > README.md

mkdir my_project
touch my_project/__init__.py
```

ローカルGitリポジトリを作成します。
名前、メールアドレスは適宜変更してください。
GitホスティングサービスとしてGitHubを使っていて、メールアドレスを公開したくない場合、
設定ページに記載されているダミーのメールアドレスが利用できます。

- [GitHub Email settings](https://github.com/settings/emails)
  - `Keep my email addresses private`の項目を参照

```shell
git init

git config user.name "John Doe"
git config user.email "mail@example.com"

git commit -m "Initial Commit" --allow-empty
```

## 開発を支援するパッケージのインストール

基本的なリンター、フォーマッター、テストツールのPythonパッケージを開発環境向けとして、プロジェクトに追加します。

```shell
poetry add --group dev pysen black isort flake8 flake8-bugbear mypy pytest
```

記事作成時点のバージョン

- pysen 0.10.5: [GitHub pfnet/pysen](https://github.com/pfnet/pysen), [PyPI pysen](https://pypi.org/project/pysen/)
  - black, isort, flake8のラッパー
- black 23.7.0: [GitHub psf/black](https://github.com/psf/black), [PyPI black](https://pypi.org/project/black/)
  - 「Python公式のスタイルガイド [PEP 8](https://peps.python.org/pep-0008/)」に基づくフォーマッター
- isort 5.12.0: [GitHub PyCQA/isort](https://github.com/PyCQA/isort), [PyPI isort](https://pypi.org/project/isort/)
  - `import`をソートするフォーマッター
- flake8 6.1.0: [GitHub PyCQA/flake8](https://github.com/PyCQA/flake8), [PyPI flake8](https://pypi.org/project/flake8/)
  - いくつかのリンターのラッパー
- flake8-bugbear 23.7.10: [GitHub PyCQA/flake8-bugbear](https://github.com/PyCQA/flake8-bugbear), [PyPI flake8-bugbear](https://pypi.org/project/flake8-bugbear/)
  - flake8のプラグイン、バグの原因になりやすい記述を見つけてくれる
- mypy 1.4.1: [GitHub python/mypy](https://github.com/python/mypy), [PyPI mypy](https://pypi.org/project/mypy/)
- pytest 7.4.0: [GitHub pytest-dev/pytest](https://github.com/pytest-dev/pytest), [PyPI pytest](https://pypi.org/project/pytest/)

プロジェクトに追加したPythonパッケージをインストールします。

```shell
poetry install
```

`pyproject.toml`に以下のように追記して、pysenの設定をします。

- [Quickstart: Set up linters using pysen](https://github.com/pfnet/pysen/blob/0.10.5/README.md#quickstart-set-up-linters-using-pysen)

```toml
[tool.pysen]
version = "0.10"

[tool.pysen.lint]
enable_black = true
enable_flake8 = true
enable_isort = true
enable_mypy = true
mypy_preset = "strict"
line_length = 88
py_version = "py311"

  [[tool.pysen.lint.mypy_targets]]
    paths = ["."]
```

pysenでは、リンター・フォーマッターは、Gitリポジトリが管理しているファイルだけに適用されます。
Gitリポジトリが管理していないファイルを扱わせたい場合は、少なくともステージングしておく必要がある点に注意してください。

次のように、リンター・フォーマッターを適用します。

```shell
git add .

poetry run pysen run lint
poetry run pysen run format
```

## 依存パッケージのインストール

### PyPIに登録されたパッケージのインストール

```shell
poetry add requests
```

### PyPI以外のパッケージリポジトリに登録されたパッケージのインストール

```shell
poetry source add --priority=explicit pytorch-cu118 "https://download.pytorch.org/whl/cu118"
poetry add --source pytorch-cu118 torch torchvision torchaudio
```

### Gitリポジトリで管理されたパッケージをインストール

```shell
poetry add git+https://github.com/org/repo.git#commithash

poetry add git+https://github.com/org/repo.git#commithash&subdirectory=subdir
```

### 開発時だけ使うパッケージのインストール

```shell
poetry add --group dev types-requests
```

### requirements.txtの出力

Poetryが管理する依存関係に基づいて、`requirements.txt`を作成します。
Poetryを使わずに実行する場合や、Dockerイメージを作る場合に有用です。
インストール先の環境が限定されるのを避けるため、ハッシュ値を含めないようにするオプション`--without-hashes`を指定しています。

```shell
poetry export --without-hashes -o requirements.txt
poetry export --without-hashes --with dev -o requirements-dev.txt
```

## Dockerfileの作成

Dockerイメージは様々な作り方が考えられますが、一例として紹介します。

### CPUだけ使う場合

```dockerfile
# syntax=docker/dockerfile:1.5
FROM python:3.11

ARG DEBIAN_FRONTEND=noninteractive
ARG PIP_NO_CACHE_DIR=1
ENV PYTHONUNBUFFERED=1
ENV PATH=/home/user/.local/bin:${PATH}

RUN <<EOF
    set -eu

    apt-get update

    apt-get install -y \
        gosu

    apt-get clean
    rm -rf /var/lib/apt/lists/*
EOF

RUN <<EOF
    set -eu

    groupadd -o -g 1000 user
    useradd -m -o -u 1000 -g user user
EOF

ADD ./requirements.txt /tmp/
RUN <<EOF
    set -eu

    gosu user pip install -r /tmp/requirements.txt
EOF

ADD ./scripts /code
ADD ./my_project /code/my_project

# 引数を受け付けない場合（環境変数や設定ファイルで設定する場合、docker compose up -dでの実行を想定する場合）
CMD [ "gosu", "user", "python", "/code/main.py" ]

# FastAPI + uvicorn（docker compose up -dでの実行を想定する場合）
# CMD [ "gosu", "user", "uvicorn", "main:app", "--host", "0.0.0.0", "--port", "5000" ]

# main.pyが引数を受け付ける場合（docker runコマンドやdocker compose runでの実行を想定する場合）
# ENTRYPOINT [ "gosu", "user", "python", "/code/main.py" ]
```

### NVIDIA GPUを使う場合

## GitHub Actions Workflowの作成

TBW。気が向いたら書きます。

### リンターによる静的検査

### PyInstallerによるバイナリビルド・リリース

### Dockerイメージのビルド・デプロイ

## GitLab CI Pipelineの作成

TBW。気が向いたら書きます。

### リンターによる静的検査

### PyInstallerによるバイナリビルド・リリース

### Dockerイメージのビルド・デプロイ
