---
canonical_url: ./
title: Ubuntu/Raspberry Pi/Debianの時刻ずれを解消する
# og_image:
# twitter_card: summary_large_image
og_description: テンプレート
date: '2021-04-04 06:00:00'
draft: false
category: Ubuntu
tags:
  - Ubuntu
  - NTP
---

# Ubuntu/Raspberry Pi/Debianの時刻ずれを解消する

```shell
$ sudo apt update
Get:1 http://raspbian.raspberrypi.org/raspbian buster InRelease [15.0 kB]
Get:2 http://archive.raspberrypi.org/debian buster InRelease [32.9 kB]
Reading package lists... Done  
E: Release file for http://raspbian.raspberrypi.org/raspbian/dists/buster/InRelease is not valid yet (invalid for another 27d 15h 31min 39s). Updates for this repository will not be applied.
E: Release file for http://archive.raspberrypi.org/debian/dists/buster/InRelease is not valid yet (invalid for another 26d 5h 35min 24s). Updates for this repository will not be applied.
```

こういうエラーが出ることがある（ログはRaspberry Pi OS）。

```shell
sudo timedatectl set-timezone Asia/Tokyo
sudo date --set 2021-04-04
sudo date --set 06:00
```

まずは手動でタイムゾーンと時刻を合わせる（だいたいでOK）。

これでaptが実行できるようになるので、Chronyを導入する。

```shell
sudo apt update
sudo apt install chrony
```

`/etc/chrony/chrony.conf`を開く。

NTP DoS対策でポートがふさがれていて、組織のNTPサーバが提供されている場合、
このようにNTPサーバを設定してあげればOK。

```shell
# pool 2.debian.pool.ntp.org iburst
pool your.ntp.server.example iburst
```

chronyを再起動。

```shell
sudo systemctl restart chrony
```


- [DateTime - Debian Wiki](https://wiki.debian.org/DateTime)
- [タイムゾーンを日本標準時(JST)に変更する CentOS 8, 7, 6 – CentOSサーバ構築術 文具堂](https://centos.bungu-do.jp/archives/67)
