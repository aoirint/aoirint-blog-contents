---
title: 'Windows シンボリックリンクを作成する'
date: '2022-02-07T12:12:18+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Windows
tags:
  - Windows
  - シンボリックリンク
---
# Windows シンボリックリンクを作成する

管理者cmdで実行する。

```cmd
; ファイル
mklink C:\link C:\original

; ディレクトリ
mklink /D C:\link C:\original
```
