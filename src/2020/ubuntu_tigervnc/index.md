---
# moved from https://aoirint.hatenablog.com/entry/2020/03/18/185309
title: Ubuntu, TigerVNC
date: '2020-03-18T18:53:09+09:00'
draft: false
channel: 技術ノート
category: Remote
tags:
- Remote
- Ubuntu
---
# Ubuntu, TigerVNC

Setup VNC via ssh on Ubuntu Desktop.

Install TigerVNC.

```sh
# Ubuntu 18.04
sudo apt install tigervnc-scraping-server

# Ubuntu 16.04
sudo apt install tigervncserver
```

Set password.

```sh
vncpasswd
```

startVNC.sh

```sh
#!/bin/bash
x0tigervncserver PasswordFile=~/.vnc/passwd Display=:0
```

configure `LocalForward LOCALPORT localhost:5900` in ~/.ssh/config.

- TigerVNCのクライアントはあんまり機能が多くない
- RealVNC社のクライアントはWindows, Linux, Mac, Mobileにリリースされてて便利
  - [https://www.realvnc.com/en/connect/download/viewer/](https://www.realvnc.com/en/connect/download/viewer/)

---

- [つまずかない！？TigerVNCでリモートデスクトップ - Qiita](https://qiita.com/Tats_U_/items/c170f61a5e03ae045128)
