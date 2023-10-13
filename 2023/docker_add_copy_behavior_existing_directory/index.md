---
title: 'Dockerfileでイメージ内の既存ディレクトリ宛にADD/COPYした場合の挙動を調べた'
date: '2023-10-13T12:50:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Docker
tags:
  - Docker
---
# Dockerfileでイメージ内の既存ディレクトリ宛にADD/COPYした場合の挙動を調べた

- Docker Engine 24.0

Dockerイメージのビルド時に、イメージ内のディレクトリ構造に、同じディレクトリ構造をもつホスト側ディレクトリを追加した場合の挙動を確認したい。
`cp -r`や`rsync -a`のような挙動を期待するが、動作を検証してみた。

結果として、`cp -r`や`rsync -a`のように、既存のディレクトリ内容を維持して、新しいファイルを追加し、重複するファイルがあれば上書きする挙動をした。

- [ADD - Dockerfile reference | Docker Docs](https://docs.docker.com/engine/reference/builder/#add)

## ADDでファイルが重複しない場合

イメージ内に以下のようなディレクトリ構造を構築する。

- /hoge
  - fuga
  - piyo/
    - hogera

```dockerfile
RUN <<EOF
    set -eu

    mkdir /hoge
    touch /hoge/fuga
    mkdir /hoge/piyo
    touch /hoge/piyo/hogera
EOF
```

ビルドコンテキストディレクトリに以下のようなディレクトリ構造を構築する。
このディレクトリを先ほどのイメージ内の`/hoge`にADDする。

- hoge/
  - fugera
  - piyo/
    - hogerara

```dockerfile
ADD ./hoge /hoge/
```

結果表示用のtreeコマンドをインストールするコマンドを加えて合わせると、以下のようなDockerfileになる。

```dockerfile
# syntax=docker/dockerfile:1.6
FROM ubuntu:22.04

RUN <<EOF
    apt-get update
    apt-get install -y \
        tree
    apt-get clean
    rm -rf /var/lib/apt/lists/*
EOF

RUN <<EOF
    set -eu

    mkdir /hoge
    touch /hoge/fuga
    mkdir /hoge/piyo
    touch /hoge/piyo/hogera
EOF

ADD ./hoge /hoge/
```

```shell
docker build -t doco .
docker run --rm -it doco
```

```plain
# tree /hoge
/hoge
|-- fuga
|-- fugara
`-- piyo
    |-- hogera
    `-- hogerara

1 directory, 4 files
```

既存のディレクトリの内容を維持したまま、新しいファイルが追加される。

## ADDでファイルが重複する場合

先ほどのビルドコンテキストディレクトリに`fuga`ファイルを追加して、以下のように適当な内容を記述する。

```plain
AAAAAAAAAAAAAAAAAAAAAAAAAAA
```

- hoge/
  - fuga
  - fugera
  - piyo/
    - hogerara

```shell
docker build -t doco .
docker run --rm -it doco
```

```plain
# cat /hoge/fuga
AAAAAAAAAAAAAAAAAAAAAAAAAAA
```

ファイルが重複する場合、上書きされる。

## COPYの場合

Dockerfileを以下のように書き換える。

```dockerfile
COPY ./hoge /hoge/
```

```shell
docker run --rm -it doco
```

```plain
# tree /hoge
/hoge
|-- fuga
|-- fugara
`-- piyo
    |-- hogera
    `-- hogerara

1 directory, 4 files
# cat /hoge/fuga
AAAAAAAAAAAAAAAAAAAAAAAAAAA
```

ADDと同じく、既存のディレクトリ内容を維持して、新しいファイルを追加し、重複するファイルがあれば上書きされた。
