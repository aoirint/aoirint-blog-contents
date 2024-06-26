---
title: 'Pythonプロジェクトの作成（pyenv + Poetry）'
date: '2023-08-07T15:45:00+09:00'
updated: '2024-04-08T21:45:00+09:00'
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

## バージョン情報

- [pyenv 2.4.0](https://github.com/pyenv/pyenv)
- [Poetry 1.8.2](https://python-poetry.org/docs/#installation)
- [Python 3.11.9](https://www.python.org/downloads/)

## 定義・ディレクトリ構成

説明のため、プロジェクトディレクトリ名`my_project`、パッケージ名`my-project`、主要なモジュール名`my_project`とします。

以下のようなディレクトリ構成にすることを想定しています。

```plain
- my_project/
  - pyproject.toml
  - Dockerfile
  - my_project/
    - __init__.py
    - __main__.py
    - cli.py
  - tests/
    - __init__.py
    - test_my_project.py
```

## Python/Poetryのインストール

pyenvでPythonをインストールします。
記事作成時点で最新のリビジョン（`0.0.x`）を記載していますが、適宜新しいバージョンが出ているか確認し、
更新してください。
マイナーバージョン（`0.x.0`）を変更する場合、依存する予定のライブラリが動作するかなど、プロジェクトの要件と相談してください。

```shell
env PYTHON_CONFIGURE_OPTS="--enable-shared" pyenv install 3.11.9
```

`PYTHON_CONFIGURE_OPTS="--enable-shared"`は、PyInstallerが動作するようにするために設定しています。

- [pyenv and PyInstaller — PyInstaller 6.5.0 documentation](https://pyinstaller.org/en/stable/development/venv.html)

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
pyenv local 3.11.9
```

現在のディレクトリにPoetryプロジェクトを作成します。
対話形式でプロジェクトの設定（`pyproject.toml`の作成）をします。

```shell
poetry init
```

（おすすめ）最後に依存ライブラリを聞かれますが、個人的には、操作がややこしく間違ったライブラリをインストールしてしまうのが怖いので、いったんスキップして後で設定することが多いです。

### Pythonパッケージ名の仕様

パッケージ名には、以下の文字が使用できます。

- ラテン文字（A-Z、大文字・小文字は区別されない）
- アラビア数字（0-9）
- ピリオド、アンダースコア、ハイフン（これらは区別されない、先頭または末尾に使用できない）

パッケージ名の仕様について、記事作成時点では以下が参考になります。

- [パッケージ名に関するPyPA仕様](https://packaging.python.org/en/latest/specifications/core-metadata/#name)
- [パッケージ名の正規化に関するPyPA仕様](https://packaging.python.org/en/latest/specifications/name-normalization/#name-normalization)
- [PEP 566 – Metadata for Python Software Packages 2.1](https://peps.python.org/pep-0566/)
- [PEP 508 – Dependency specification for Python Software Packages](https://peps.python.org/pep-0508/)

### .gitignoreの作成

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

### 必須ファイルの作成

主要なドキュメント`README.md`、主要なモジュールのディレクトリ`my_project/`、ファイル`my_project/__init__.py`を作成します。
これらのファイルは、プロジェクトに対してPoetryを動作させるために必要です（Poetry実行時にファイルが存在しない旨のエラーが出ます）。

```shell
echo "# my_project" > README.md

mkdir my_project
touch my_project/__init__.py
```

### メインスクリプトの作成

#### `my_project/__init__.py`

```python
__version__ = "0.0.0"
```

#### `my_project/___main__.py`

```python
from .cli import main

if __name__ == "__main__":
    main()
```

#### `my_project/cli.py`

```python
import logging
from argparse import ArgumentParser, Namespace
from logging import getLogger

from . import __version__ as APP_VERSION

logger = getLogger(__name__)


def execute_command(
    args: Namespace,
) -> None:
    pass


def execute_subcommand_mysubcommand(
    args: Namespace,
) -> None:
    pass


def main() -> None:
    parser = ArgumentParser()
    parser.add_argument(
        "--version",
        action="version",
        version=APP_VERSION,
    )
    parser.set_defaults(handler=execute_command)

    subparsers = parser.add_subparsers()

    subparser_mysubcommand = subparsers.add_parser("mysubcommand")
    subparser_mysubcommand.set_defaults(handler=execute_subcommand_mysubcommand)

    logging.basicConfig(
        level=logging.INFO,
        format="%(asctime)s %(levelname)s %(name)s : %(message)s",
    )

    args = parser.parse_args()
    if hasattr(args, "handler"):
        args.handler(args)
    else:
        parser.print_help()
```

### Gitリポジトリの作成

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

この記事で想定しているバージョン情報

- pysen 0.11.0: [GitHub pfnet/pysen](https://github.com/pfnet/pysen), [PyPI pysen](https://pypi.org/project/pysen/)
  - black, isort, flake8のラッパー
- black 24.3.0: [GitHub psf/black](https://github.com/psf/black), [PyPI black](https://pypi.org/project/black/)
  - 「Python公式のスタイルガイド [PEP 8](https://peps.python.org/pep-0008/)」に基づくフォーマッター
- isort 5.13.2: [GitHub PyCQA/isort](https://github.com/PyCQA/isort), [PyPI isort](https://pypi.org/project/isort/)
  - `import`をソートするフォーマッター
- flake8 7.0.0: [GitHub PyCQA/flake8](https://github.com/PyCQA/flake8), [PyPI flake8](https://pypi.org/project/flake8/)
  - いくつかのリンターのラッパー
- flake8-bugbear 24.2.6: [GitHub PyCQA/flake8-bugbear](https://github.com/PyCQA/flake8-bugbear), [PyPI flake8-bugbear](https://pypi.org/project/flake8-bugbear/)
  - flake8のプラグイン、バグの原因になりやすい記述を見つけてくれる
- mypy 1.9.0: [GitHub python/mypy](https://github.com/python/mypy), [PyPI mypy](https://pypi.org/project/mypy/)
- pytest 8.1.1: [GitHub pytest-dev/pytest](https://github.com/pytest-dev/pytest), [PyPI pytest](https://pypi.org/project/pytest/)

プロジェクトに追加したPythonパッケージをインストールします。

```shell
poetry install
```

`pyproject.toml`に以下のように追記して、pysenの設定をします。

- [Quickstart: Set up linters using pysen](https://github.com/pfnet/pysen/blob/0.10.5/README.md#quickstart-set-up-linters-using-pysen)

```toml
[tool.pysen]
version = "0.11"

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
# PyTorch for CUDA 11.8
poetry source add --priority=explicit pytorch-cu118 "https://download.pytorch.org/whl/cu118"
poetry add --source pytorch-cu118 torch torchvision torchaudio

# PyTorch for CUDA 12.1
poetry source add --priority=explicit pytorch-cu121 "https://download.pytorch.org/whl/cu121"
poetry add --source pytorch-cu121 torch torchvision torchaudio
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

NOTE: この項目で扱うエクスポート機能は、Poetry 1.2からPoetry 1.7までPoetry本体に実装されていましたが、
Poetry 1.8からプラグイン`poetry-plugin-export`として分離されました。

Poetry 1.8では`poetry-plugin-export`がPoetryの実行環境にデフォルトでインストールされているため、
Poetry 1.7までと同じ挙動が維持されていますが、今後のアップデートでデフォルトではインストールされなくなります。
以下のコマンドで明示的にインストールしておきましょう。

- Poetry 1.8.0のリリースノート: [Upcoming Changes: Removing poetry-plugin-export from the default installation](https://python-poetry.org/blog/announcing-poetry-1.8.0/#removing-poetry-plugin-export-from-the-default-installation)

```shell
poetry self add poetry-plugin-export

# exportコマンド実行時に上記内容を説明する警告の表示を無効化
poetry config warnings.export false
```

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

<details>

```dockerfile
# syntax=docker/dockerfile:1.6
FROM python:3.11

ARG DEBIAN_FRONTEND=noninteractive
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

ARG CONTAINER_UID=1000
ARG CONTAINER_GID=1000
RUN <<EOF
    set -eu

    groupadd --non-unique --gid "${CONTAINER_GID}" user
    useradd --non-unique --uid "${CONTAINER_UID}" --gid "${CONTAINER_GID}" --create-home user
EOF

ADD ./requirements.txt /tmp/
RUN --mount=type=cache,uid=${CONTAINER_UID},gid=${CONTAINER_GID},target=/home/user/.cache/pip <<EOF
    set -eu

    gosu user pip install -r /tmp/requirements.txt
EOF

ADD ./pyproject.toml ./README.md /code/my_project/
ADD ./my_project /code/my_project/my_project

RUN --mount=type=cache,uid=${CONTAINER_UID},gid=${CONTAINER_GID},target=/home/user/.cache/pip <<EOF
    set -eu

    gosu user pip install -e /code/my_project
EOF

RUN <<EOF
    set -eu

    mkdir -p /work
    chown -R "${CONTAINER_UID}:${CONTAINER_GID}" /work
EOF

WORKDIR /work

# 引数を受け付けない場合（環境変数や設定ファイルで設定する場合、docker compose up -dでの実行を想定する場合）
CMD [ "gosu", "user", "python", "-m", "my_project" ]

# main.pyが引数を受け付ける場合（docker runコマンドやdocker compose runでの実行を想定する場合）
# ENTRYPOINT [ "gosu", "user", "python", "-m", "my_project" ]
```

</details>

### NVIDIA GPUを使う場合

||--build-arg BASE_RUNTIME_IMAGE|リポジトリ|
|:--|:--|:--|
|CPU|`ubuntu:22.04`|[Docker Hub: ubuntu](https://hub.docker.com/_/ubuntu)|
|NVIDIA Driver v525|`nvcr.io/nvidia/driver:525-signed-ubuntu22.04`|[NGC: NVIDIA GPU Driver](https://catalog.ngc.nvidia.com/orgs/nvidia/containers/driver)|
|CUDA 11.8 + cuDNN 8|`nvidia/cuda:11.8.0-cudnn8-runtime-ubuntu22.04`|[Docker Hub: nvidia/cuda](https://hub.docker.com/r/nvidia/cuda)|
|CUDA 11.8 + cuDNN 8（開発用ライブラリ入）|`nvidia/cuda:11.8.0-cudnn8-devel-ubuntu22.04`|[Docker Hub: nvidia/cuda](https://hub.docker.com/r/nvidia/cuda)|

ビルド環境、実行環境に[NVIDIA Container Toolkitのインストール](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html)が必要です。

ビルド時、`docker build --build-arg BASE_RUNTIME_IMAGE=nvcr.io/nvidia/driver:525-signed-ubuntu22.04`のようにベースイメージを切り替える想定のDockerfileになっています。

Dockerイメージ実行時、GPUを使用するには`docker run --gpus all`のように`--gpus`オプションでGPUの使用を明示する必要があります。

<details>

```dockerfile
# syntax=docker/dockerfile:1.6
ARG BASE_IMAGE=ubuntu:22.04
ARG BASE_RUNTIME_IMAGE=${BASE_IMAGE}

FROM ${BASE_IMAGE} AS python-env

ARG DEBIAN_FRONTEND=noninteractive
ARG PIP_NO_CACHE_DIR=1
ENV PYTHONUNBUFFERED=1

ARG PYENV_VERSION=v2.4.0
ARG PYTHON_VERSION=3.11.9

RUN <<EOF
    set -eu

    apt-get update

    apt-get install -y \
        make \
        build-essential \
        libssl-dev \
        zlib1g-dev \
        libbz2-dev \
        libreadline-dev \
        libsqlite3-dev \
        wget \
        curl \
        llvm \
        libncursesw5-dev \
        xz-utils \
        tk-dev \
        libxml2-dev \
        libxmlsec1-dev \
        libffi-dev \
        liblzma-dev \
        git

    apt-get clean
    rm -rf /var/lib/apt/lists/*
EOF

RUN <<EOF
    set -eu

    git clone https://github.com/pyenv/pyenv.git /opt/pyenv
    cd /opt/pyenv
    git checkout "${PYENV_VERSION}"

    PREFIX=/opt/python-build /opt/pyenv/plugins/python-build/install.sh
    /opt/python-build/bin/python-build -v "${PYTHON_VERSION}" /opt/python

    rm -rf /opt/python-build /opt/pyenv
EOF


FROM ${BASE_RUNTIME_IMAGE} AS runtime-env

ARG DEBIAN_FRONTEND=noninteractive
ENV PYTHONUNBUFFERED=1
ENV PATH=/home/user/.local/bin:/opt/python/bin:${PATH}

RUN <<EOF
    set -eu

    apt-get update

    apt-get install -y \
        gosu

    apt-get clean
    rm -rf /var/lib/apt/lists/*
EOF

ARG CONTAINER_UID=1000
ARG CONTAINER_GID=1000
RUN <<EOF
    set -eu

    groupadd --non-unique --gid "${CONTAINER_GID}" user
    useradd --non-unique --uid "${CONTAINER_UID}" --gid "${CONTAINER_GID}" --create-home user
EOF

COPY --from=python-env /opt/python /opt/python

ADD ./requirements.txt /tmp/
RUN --mount=type=cache,uid=${CONTAINER_UID},gid=${CONTAINER_GID},target=/home/user/.cache/pip <<EOF
    set -eu

    gosu user pip install -r /tmp/requirements.txt
EOF

ADD ./pyproject.toml ./README.md /code/my_project/
ADD ./my_project /code/my_project/my_project

RUN --mount=type=cache,uid=${CONTAINER_UID},gid=${CONTAINER_GID},target=/home/user/.cache/pip <<EOF
    set -eu

    gosu user pip install -e /code/my_project
EOF

RUN <<EOF
    set -eu

    mkdir -p /work
    chown -R "${CONTAINER_UID}:${CONTAINER_GID}" /work
EOF

WORKDIR /work

# 引数を受け付けない場合（環境変数や設定ファイルで設定する場合、docker compose up -dでの実行を想定する場合）
CMD [ "gosu", "user", "python", "-m", "my_project" ]

# main.pyが引数を受け付ける場合（docker runコマンドやdocker compose runでの実行を想定する場合）
# ENTRYPOINT [ "gosu", "user", "python", "-m", "my_project" ]
```

</details>

## GitHub Actions Workflowの作成

### GitHub Actions リンターによる静的検査

<details>

```yaml
# lint.yml
name: Lint

on:
  push:
  pull_request:
  workflow_dispatch:

env:
  PYTHON_VERSION: '3.11.9'

jobs:
  lint:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Install Poetry
        shell: bash
        run: pipx install poetry

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "${{ env.PYTHON_VERSION }}"
          cache: 'poetry'

      - name: Install Dependencies
        shell: bash
        run: poetry install

      - name: Run lint
        shell: bash
        run: poetry run pysen run lint
```

</details>

### GitHub Actions PyInstallerによるバイナリビルド・リリース

### GitHub Actions Dockerイメージのビルド・リリース

GitHub Variablesに`DOCKERHUB_USERNAME`を設定し、GitHub Secretsに`DOCKERHUB_TOKEN`を設定する必要があります。

`my_project/__init__.py`に`__version__ = "0.0.0"`を記述し、
`pyproject.toml`に`version = "0.0.0"`を記述します。
これらのバージョンは、開発中は`0.0.0`となり、リリース時はリリースバージョンに置換されます。

#### CPUだけ使うDockerfileの場合

<details>

```yaml
# build-docker.yml
name: Build Docker

on:
  push:
    branches:
      - main
  release:
    types:
      - created
  workflow_dispatch:

env:
  IMAGE_NAME: aoirint/my_project
  IMAGE_TAG: ${{ github.event.release.tag_name != '' && github.event.release.tag_name || 'latest' }}
  VERSION: ${{ (github.event.release.tag_name != '' && github.event.release.tag_name) || '0.0.0' }}

jobs:
  docker-build-and-push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Setup Docker Buildx
        id: buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Registry
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Replace Version
        shell: bash
        run: |
          sed -i "s/__version__ = \"0.0.0\"/__version__ = \"${{ env.VERSION }}\"/" my_project/__init__.py
          sed -i "s/version = \"0.0.0\"/version = \"${{ env.VERSION }}\"/" pyproject.toml

      - name: Build and Deploy Docker image
        uses: docker/build-push-action@v5
        env:
          IMAGE_NAME_AND_TAG: ${{ format('{0}:{1}', env.IMAGE_NAME, env.IMAGE_TAG) }}
          IMAGE_CACHE_FROM: ${{ format('type=registry,ref={0}:latest-buildcache', env.IMAGE_NAME) }}
          IMAGE_CACHE_TO: ${{ env.IMAGE_TAG == 'latest' && format('type=registry,ref={0}:latest-buildcache,mode=max', env.IMAGE_NAME) || '' }}
        with:
          context: .
          builder: ${{ steps.buildx.outputs.name }}
          file: ./Dockerfile
          push: true
          tags: ${{ env.IMAGE_NAME_AND_TAG }}
          cache-from: ${{ env.IMAGE_CACHE_FROM }}
          cache-to: ${{ env.IMAGE_CACHE_TO }} 
```

</details>

#### CPU版イメージとGPU版イメージをビルドする場合

<details>

```yaml
# build-docker.yml
name: Build Docker

on:
  push:
    branches:
      - main
  release:
    types:
      - created
  workflow_dispatch:

env:
  IMAGE_NAME: aoirint/my_project
  IMAGE_VERSION_NAME: ${{ (github.event.release.tag_name != '' && github.event.release.tag_name) || 'latest' }}
  VERSION: ${{ (github.event.release.tag_name != '' && github.event.release.tag_name) || '0.0.0' }}
  PYTHON_VERSION: '3.11.9'

jobs:
  docker-build-and-push:
    strategy:
      fail-fast: false
      matrix:
        include:
          -
            base_image: 'ubuntu:22.04'
            base_runtime_image: 'ubuntu:22.04'
            image_variant_name: 'ubuntu'
          -
            base_image: 'ubuntu:22.04'
            base_runtime_image: 'nvcr.io/nvidia/driver:525-signed-ubuntu22.04'
            image_variant_name: 'nvidia'

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3

      - name: Setup Docker Buildx
        id: buildx
        uses: docker/setup-buildx-action@v3

      - name: Login to Docker Registry
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Replace Version
        shell: bash
        run: |
          sed -i "s/__version__ = \"0.0.0\"/__version__ = \"${{ env.VERSION }}\"/" my_project/__init__.py
          sed -i "s/version = \"0.0.0\"/version = \"${{ env.VERSION }}\"/" pyproject.toml

      - name: Build and Deploy Docker image
        uses: docker/build-push-action@v5
        env:
          IMAGE_NAME_AND_TAG: ${{ format('{0}:{1}-{2}', env.IMAGE_NAME, matrix.image_variant_name, env.IMAGE_VERSION_NAME) }}
          LATEST_IMAGE_NAME_AND_TAG: ${{ format('{0}:{1}-{2}', env.IMAGE_NAME, matrix.image_variant_name, 'latest') }}
        with:
          builder: ${{ steps.buildx.outputs.name }}
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ env.IMAGE_NAME_AND_TAG }}
          build-args: |
            BASE_IMAGE=${{ matrix.base_image }}
            BASE_RUNTIME_IMAGE=${{ matrix.base_runtime_image }}
            PYTHON_VERSION=${{ env.PYTHON_VERSION }}
          target: runtime-env
          cache-from: |
            type=registry,ref=${{ env.IMAGE_NAME_AND_TAG }}-buildcache
            type=registry,ref=${{ env.LATEST_IMAGE_NAME_AND_TAG }}-buildcache
          cache-to: |
            type=registry,ref=${{ env.IMAGE_NAME_AND_TAG }}-buildcache,mode=max
```

</details>

## GitLab CI Pipelineの作成

TBW。気が向いたら書きます。

### GitLab CI リンターによる静的検査

### GitLab CI PyInstallerによるバイナリビルド・リリース

### GitLab CI Dockerイメージのビルド・リリース
