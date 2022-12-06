---
title: 'Ubuntu 20.04, ネットワーク設定変更時の認証を回避する'
date: '2021-11-29T20:30:00+09:00'
updated: '2021-11-29T20:30:00+09:00'
draft: false
channel: 技術ノート
category: Ubuntu
tags:
  - Ubuntu
  - Network
---

# Ubuntu 20.04, ネットワーク設定変更時の認証を回避する

Ubuntu 20.04にアップデートしてから、ネットワーク設定の変更時（接続のON/OFFなど）に`System policy prevents control of network connections`と表示され、認証を要求されるようになりました。

ここまではまだいいのですが、認証が表示されるたびに数秒間デスクトップがハングする状態になっていて、ストレスでした。
Keyringのロックを解除するときも数秒間ハングするので、似た原因がありそうです。

ひとまず、設定変更時の認証を要求されないようにして、ハングを防ぐことを試みました。結果として試みは成功し、ハングを回避できるようになりました。

- [https://unix.stackexchange.com/questions/534469/system-policy-prevents-control-of-network-connections](https://unix.stackexchange.com/questions/534469/system-policy-prevents-control-of-network-connections)
- [https://code.luasoftware.com/tutorials/linux/ubuntu-prompt-system-policy-prevents-modification-of-network-settings-for-all-users/](https://code.luasoftware.com/tutorials/linux/ubuntu-prompt-system-policy-prevents-modification-of-network-settings-for-all-users/)

## /etc/polkit-1/localauthority/50-local.d/50-allow-network-manager.pkla

```pkla
[Allow Network Manager all Users]
Identity=unix-user:*
Action=org.freedesktop.NetworkManager.settings.modify.system;org.freedesktop.NetworkManager.network-control
ResultAny=no
ResultInactive=no
ResultActive=yes
```

```shell
systemctl restart network-manager.service
```
