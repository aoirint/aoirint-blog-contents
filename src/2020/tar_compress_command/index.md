---
# moved from https://aoirint.hatenablog.com/entry/2020/03/17/143710
title: tar 圧縮
date: '2020-03-17T14:37:10+09:00'
draft: false
channel: 技術ノート
category: Ubuntu
tags:
- Ubuntu
---
# tar 圧縮

解凍`tar -xf`はよく使うので覚えるけど、圧縮のほうが覚えられん..

- [[Linux]ファイルの圧縮、解凍方法 - Qiita](https://qiita.com/supersaiakujin/items/c6b54e9add21d375161f)

tar.gz

```sh
tar -zcf compressed.tar.gz FILES
```

extractとcompressかな

おまけ

zip

```sh
zip -r compressed.zip DIR
unzip compressed.zip
```
