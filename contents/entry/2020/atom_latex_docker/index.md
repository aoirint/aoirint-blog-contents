---
title: Atom + LaTeX in Docker
# og_image:
# twitter_card: summary_large_image
og_description: AtomからLaTeX in Dockerを呼び出してTeX環境を整備する
date: '2020-09-17 08:35:00'
updated: '2021-05-14 01:00:00'
draft: false
category: LaTeX
tags:
  - Atom
  - LaTeX
  - TeXLive
  - Docker
---
# Atom + LaTeX in Docker

## 概要
LaTeXをホストにインストールせず、Docker内で動かしつつ、
Atomからこれを利用できるようにする。

## Requirements
- Ubuntu 18.04
- Docker
- Atom
- Atom Packages
    - [latex](https://atom.io/packages/latex)

## Atom Packages
[latex](https://atom.io/packages/latex)を入れれば最低限ビルドコマンドは叩けるようになる。

- [latex](https://atom.io/packages/latex)
- シンタックスハイライト
    - [language-latex](https://atom.io/packages/language-latex)
- アウトライン表示
    - [document-outline](https://atom.io/packages/document-outline)
- PDFプレビュー
    - [pdf-view]()（メモリリーク? Atomを一度閉じれば解消する）
    - または [pdf-view-plus]()（メモリリーク対策版らしい。`latex`との連携はないので注意）


## Dockerイメージ
[paperist/alpine-texlive-ja](https://hub.docker.com/r/paperist/alpine-texlive-ja/)を使う。

```
#!bash
sudo docker pull paperist/alpine-texlive-ja
```

## Docker
`sudo`なしでDockerを実行できるようにする。AtomからDockerコンテナを作るのに必要。

注意として、この方法で一般ユーザがDockerを使えるようにすると特権昇格できてしまうため、
共有サーバにおいてはDocker 20.10以降のDocker rootlessを設定する。

- [https://docs.docker.com/engine/security/rootless/](https://docs.docker.com/engine/security/rootless/)

ユーザをdockerグループに追加したあと再ログインする。新しくdockerグループが作られた直後は`newgrp docker`しなければならないことがあり、シェルごとにこれを実行する必要があるようなのでAtomに反映されず、この場合OSの再起動が必要。

```
#!bash
sudo groupadd docker
sudo adduser $USER docker
```

## latexmkスクリプトを作る

`/usr/texbin/latexmk`（手動で`latex`の`TeX Path`を設定するか、デフォルトでPATHの通っている場所ならどこでもいい）に以下のシェルスクリプトを作成し、`chmod +x /usr/texbin/latexmk`しておく。

`${HOME}/.atom/packages/latex/resources`のマウントは`latex`の`Extended Build Mode`が有効のときに`${HOME}/.atom/packages/latex/resources/latexmkrc`が読み出されるため設定している（このパスはホストのAtomから渡されるのでマウント先パスもホストと同じ）。この機能を無効にしていれば不要。

```
#!/bin/sh
docker run --rm \
  -v "${PWD}:/workdir" \
  -v "${HOME}/.atom/packages/latex/resources:${HOME}/.atom/packages/latex/resources" \
  paperist/alpine-texlive-ja \
  latexmk "$@"
```

## サンプルTeXファイル

```
#!tex
\documentclass[10pt,a4paper]{jsarticle}

\title{My Title}
\author{Author}
\date{2020-09-17}

\begin{document}
\maketitle

\section{サンプル}

\end{document}
```

## 注意点
カレントディレクトリ以下をマウントするため、外部においた`.sty`などは読み込めないので注意（デフォルトでロードされるディレクトリがあれば追加のマウントをすればOKと思われる）。

## おまけ
### stderrにコマンドを吐き出してエラー終了するスクリプト

```
#!/bin/sh
echo $0 $@ >&2
exit 1;
```
