---
# moved from https://aoirint.hatenablog.com/entry/2018/08/29/040907
title: 'SSD換装時のOS移動（スケールダウン）'
date: '2018-08-29T04:09:07+09:00'
draft: false
channel: 技術ノート
category: ストレージ
tags:
  - 'ストレージ'
  - 'OS'
---
# SSD換装時のOS移動（スケールダウン）

容量の小さなSSDに換装する（HDD 500GBからSSD 250GB）。

1. 対象のHDD, SSD以外でUbuntuを起動（USBブートなど）
2. GPartedで必要なパーティションをHDDからSSDにコピー（Boot, OSとか）
3. コピー先でパーティションを右クリック、Manage Flagsからコピー元のパーティションとFlagsを一致させる
4. Ubuntuとのデュアルブートにしていたため、EFIをマウントして"ubuntu"を削除
