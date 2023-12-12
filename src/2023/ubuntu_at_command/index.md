---
title: 'Ubuntu コマンドの実行を1度だけスケジュールする（atコマンド）'
date: '2023-06-28T19:15:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: 'Command Utility'
tags:
  - Ubuntu
  - 'Command Utility'
---
# Ubuntu コマンドの実行を1度だけスケジュールする（atコマンド）

```shell
sudo apt install at
```

コマンドは`/bin/sh`で実行される。

## 次の18:00に1度だけコマンドを実行

```shell
echo 'wondershaper clear eno1' | sudo at 18:00
```

## 2023年6月29日 18:00に1度だけコマンドを実行

```shell
echo 'wondershaper clear eno1' | sudo at 18:00 2023-06-29
```

## 実行スケジュール（ジョブ一覧）を確認する

```shell
sudo at -l
```

## ジョブ1番の実行スケジュールをキャンセルする

```shell
sudo at -d 1
```

- [https://yuya-hirooka.hatenablog.com/entry/2021/01/22/215657](https://yuya-hirooka.hatenablog.com/entry/2021/01/22/215657)
