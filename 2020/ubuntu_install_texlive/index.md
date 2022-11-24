---
# moved from https://aoirint.hatenablog.com/entry/2020/04/12/014901
title: Install TexLive on Ubuntu 18.04
date: '2020-04-12 01:49:01'
draft: false
channel: 技術ノート
category: TeX
tags:
- TeX
- Ubuntu
---
# Install TexLive on Ubuntu 18.04

普段TeXを書くときはAtom + latex + pdf-viewなのですが、うまくpdf-viewが動作してくれなくなった（空のタブが開いたりする）ので検証ついでに（関係ないとは思いつつ）TeXのバージョンを上げてみたくなった。aptからインストーラでのインストールに変更。

TeXはインストールに時間がかかっていけない..。dockerイメージ化したいが、毎回dockerコンテナ付けるとめっちゃオーバーヘッドできそう。`up -d`して`exec`で動かせば速いのかな？　一般ユーザでdockerコマンド実行できるようにもしないといけないか。

```sh
sudo apt purge texlive-*
```

東京周辺ならJAIST、山形大学などのミラーサーバに繋がるかと思います。

```sh
wget http://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz
```

- SHA-512 (2020-04-11)
- [https://ctan.org/tex-archive/systems/texlive/tlnet](https://ctan.org/tex-archive/systems/texlive/tlnet)

```sh
sha512sum install-tl-unx.tar.gz
6fbca1895f0f08645e2ff97b34b9b07d659af7d25de1edc6e123fc472f3dd05db42edc9e5450fd09fae548f5259f93d7f98f63d29dfe6ff7e069685ba202661b  install-tl-unx.tar.gz
```

```sh
tar xf install-tl-unx.tar.gz
cd install-tl-*
sudo ./install-tl
```

```
Actions:
 <I> start installation to hard disk
 <P> save installation profile to 'texlive.profile' and exit
 <H> help
 <Q> quit

Enter command: I
Installing to: /usr/local/texlive/2020
Installing [0001/3978, time/total: ??:??/??:??]: texlive.infra [420k]
Installing [0002/3978, time/total: 00:01/02:06:37]: texlive.infra.x86_64-linux [143k]
```

おうちの回線では1h-2hくらい。おやすみなさい..

```
Welcome to TeX Live!


See /usr/local/texlive/2020/index.html for links to documentation.
The TeX Live web site (https://tug.org/texlive/) contains any updates and
corrections. TeX Live is a joint project of the TeX user groups around the
world; please consider supporting it by joining the group best for you. The
list of groups is available on the web at https://tug.org/usergroups.html.


Add /usr/local/texlive/2020/texmf-dist/doc/man to MANPATH.
Add /usr/local/texlive/2020/texmf-dist/doc/info to INFOPATH.
Most importantly, add /usr/local/texlive/2020/bin/x86_64-linux
to your PATH for current and future sessions.
Logfile: /usr/local/texlive/2020/install-tl.log
```

```sh
export MANPATH=/usr/local/texlive/2020/texmf-dist/doc/man:$MANPATH
export INFOPATH=/usr/local/texlive/2020/texmf-dist/doc/info:$INFOPATH
export PATH=/usr/local/texlive/2020/bin/x86_64-linux:$PATH
```

...問題は直りませんでした。やっぱ関係ないか。本題についてはエンジンをplatexからuplatexに変更してみたら解決した気がする。ついでにAtomのバージョンが古くなっていたのでアップデート。レスポンス良くなった？

- [https://texwiki.texjp.org/?Linux#texliveinstall](https://texwiki.texjp.org/?Linux#texliveinstall)
- [https://qiita.com/BitPositive/items/6b13e2038d628c33be8e](https://qiita.com/BitPositive/items/6b13e2038d628c33be8e)
- [https://qiita.com/Shitimi_613/items/9706d57fb7bc17cbed0e](https://qiita.com/Shitimi_613/items/9706d57fb7bc17cbed0e)
