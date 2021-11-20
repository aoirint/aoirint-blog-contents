---
title: Makefile Tips
date: '2021-07-01 13:00:00'
draft: false
category: Command Utility
tags:
  - Command Utility
  - Make
  - Makefile
---

# Makefile Tips

## Makefileのあるディレクトリの絶対パスを取得する

`$(pwd)`や相対パスでは、`make`コマンドの実行ディレクトリが基準になってしまう。`Makefile`の実体があるディレクトリを基準としたいときは、以下のようにROOT_DIRを定義する。

```makefile
ROOT_DIR = $(dir $(realpath $(firstword $(MAKEFILE_LIST))))
```


## エラー時に処理を続行する

コマンドの前に`-`（ハイフン）をつける。

```makefile
.PHONY: cmd
cmd:
	ls
	-fail_command a b c
	ls
```


## インデントはスペースではなくタブを使う

vimrcで`set expandtab`している場合にひっかかる。

以下で一時的に解除するか、vimrcで`set expandtab`するのをやめる、もしくはvimrcでファイル名による分岐で設定する。

```vim
:set noexpandtab
```

### .vimrc
```vim
set expandtab

let _filename = expand('%:r')
if _filename == 'Makefile'
  set noexpandtab
endif
```

- [vimでMakefileだけTabスペースではなくTabとして入力する - Qiita](https://qiita.com/Lacty/items/23a89d2b999cb0e9fae1)


## ファイルの存在に関係なくターゲットを実行する

Makeは本来Cのコンパイルなどに使うとき、ターゲット名に一致するファイルが存在している場合に処理をスキップするようになっている。

一方で、`build`というターゲット名を使って、呼び出しのたびにコマンドを実行したいが、`build`ディレクトリが存在するような場合に困る。

スキップさせないようにするには、`.PHONY: ターゲット名`を記述する。

```makefile
.PHONY: cmd
cmd:
	ls
```
