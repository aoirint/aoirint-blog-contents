---
title: 'Ubuntu ネットワーク通信量をターミナルからグラフで見る（Speedometer）'
date: '2023-05-19T09:30:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Ubuntu
tags:
  - Ubuntu
  - Network
  - 'Network Traffic'
  - Speedometer
---
# Ubuntu ネットワーク通信量をターミナルからグラフで見る（Speedometer）

- Ubuntu Server 22.04

```shell
sudo apt install speedometer
```

通信量を見たいネットワークデバイスの名前を調べる（`eno1`、`enp0s31f6`、`enp3s0`、`eth0`、`wlan0`など）。

```shell
ip addr
```

`eno1`を目的のデバイスの名前に置き換えて、以下のコマンドを実行する。

```shell
speedometer -l -r eno1 -t eno1 -m $(( 1024 * 1024 * 3 / 2 ))

# other examples
speedometer -l -r enp0s31f6 -t enp0s31f6 -m $(( 1024 * 1024 * 3 / 2 ))

speedometer -l -r enp3s0 -t enp3s0 -m $(( 1024 * 1024 * 3 / 2 ))

speedometer -l -r eth0 -t eth0 -m $(( 1024 * 1024 * 3 / 2 ))

speedometer -l -r wlan0 -t wlan0 -m $(( 1024 * 1024 * 3 / 2 ))
```

## 参考

- [networking - How to display network traffic in the terminal? - Ask Ubuntu](https://askubuntu.com/questions/257263/how-to-display-network-traffic-in-the-terminal)
