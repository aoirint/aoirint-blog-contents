---
title: Imagemagickで画像のDPIを変更する
# og_image:
# twitter_card: summary_large_image
og_description: Imagemagickで画像のDPIを変更する
date: '2020-11-09T06:30:00+09:00'
draft: false
channel: 技術ノート
category: 'Command Utility'
tags:
  - 'Command Utility'
  - '画像'
  - 'Imagemagick'
  - TeX
  - LaTeX
---

# Imagemagickで画像のDPIを変更する

画像のDPI（dot per inch）がばらばらの場合、
TeXで画像を取り込んだときのサイズがばらばらになるため統一する。

以下はIN.pngをDPI 72にしてOUT.pngに出力するコマンド。

```bash
convert -density 72 -units PixelsPerInch IN.png OUT.png
```

もともとのDPIを調べるコマンド。

```bash
identify -format '%x,%y\n' IN.png
```
