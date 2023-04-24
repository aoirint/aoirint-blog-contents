---
title: '動画を繋げる（FFmpeg）'
date: '2023-04-24T16:00:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: 動画処理
tags:
  - 動画処理
  - FFmpeg
---
# 動画を繋げる（FFmpeg）

- Git Bash (Git for Windows) 2.32.0
- ffmpeg version N-109977-gaca7ef78cc-20230309

連番の動画ファイルを順番通りに繋げて、1つの動画ファイルにします。

`ls`コマンドの`-v`オプションは、自然な数字順にソートします。

```shell
ls -v *.ts | sed "s/.*/file '&'/" > list.txt
ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp4
```

- [Concatenate – FFmpeg](https://trac.ffmpeg.org/wiki/Concatenate)
- [【 ls 】コマンド（並べ替え編）――表示結果を並べ替える：Linux基本コマンドTips（29） - ＠IT](https://atmarkit.itmedia.co.jp/ait/articles/1607/05/news023.html)
- [Add prefix to line that is the pattern of text after dot using sed or other command - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/688386/add-prefix-to-line-that-is-the-pattern-of-text-after-dot-using-sed-or-other-comm)
