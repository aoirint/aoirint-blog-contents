---
title: 'Ubuntu ネットワーク帯域を制限する（Wondershaper）'
date: '2023-06-28T19:15:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Network
tags:
  - Ubuntu
  - Network
---
# Ubuntu ネットワーク帯域を制限する（Wondershaper）

```shell
# 上下300kbpsに制限
sudo wondershaper eno1 300 300
sudo wondershaper enp0s31f6 300 300

# 下り1000kbps、上り300kbpsに制限
sudo wondershaper eno1 1000 300
sudo wondershaper enp0s31f6 1000 300

# 帯域制限を解除
sudo wondershaper clear eno1
sudo wondershaper clear enp0s31f6
```

- [https://askubuntu.com/questions/20872/how-do-i-limit-internet-bandwidth](https://askubuntu.com/questions/20872/how-do-i-limit-internet-bandwidth)

## rootユーザで/bin/shによるコマンド実行をスケジュール（atコマンド）

```shell
echo "wondershaper eno1 300 300" | sudo at 18:00 2023-06-29
echo "wondershaper clear eno1" | sudo at 2:00 2023-06-29
```
