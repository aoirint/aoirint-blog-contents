---
# moved from https://aoirint.hatenablog.com/entry/2017/09/24/094424
title: 'MSYS2でPython3を使う'
date: '2017-09-24T09:44:24+09:00'
draft: false
channel: 技術ノート
category: MSYS2
tags:
  - MSYS2
  - Windows
  - Python
---
# MSYS2でPython3を使う

## What

MSYS2でPython3を使いたい。

## Environment

Windows 10 Home

## How

`$ pacman -S python`で最新のPythonが入る。 `$ pacman -S python3-pip`でPython3のpipが入る。

例えばPython 3.6.2を入れたとして、実行するには`python`、`python3`か`python3.6`。

## Appendix

### バージョン合わせ

```shell
export PATH="/mingw64/bin:${PATH}"
```

上のように`.bash_profile`をいじってMinGWのbinをPATHに入れて動かしていると、pythonのバージョンが合わないときがある。

pythonで最新のPythonを動かしたいときは、

```shell
export PATH="${PATH}:/mingw64/bin"
```

とすれば動く。
