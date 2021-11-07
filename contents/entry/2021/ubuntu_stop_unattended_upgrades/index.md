---
title: 'Ubuntu 20.04, Unattended Upgradesを停止'
date: '2021-11-08 07:30:00'
draft: false
category: Ubuntu
tags:
  - Ubuntu
  - dpkg
  - apt
---
# Ubuntu 20.04, Unattended Upgradesを停止

- <https://unix.stackexchange.com/questions/470709/how-do-i-stop-disable-unattended-upgrades-from-being-launched-automatically>

勝手にnvidia driverの更新が走って、
カーネルに読み込まれているdriverバージョンと、
ディスク上にあるdriverバージョンが一致しなくなって、
デスクトップアプリが起動しなくなったりする。

どうせ結構な頻度でapt upgradeするし、
再起動が保証されないタイミングで更新されて、
不安定になるのはいただけない。

```shell
sudo dpkg-reconfigure -plow unattended-upgrades
```

これでTUIが起動するので、`<No>`を選ぶ。
