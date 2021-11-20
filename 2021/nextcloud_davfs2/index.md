---
title: 'Nextcloud davfs2 マウント設定（Ubuntu）'
date: '2021-11-13 17:20:00'
draft: false
category: Nextcloud
tags:
  - Nextcloud
  - Ubuntu
---
# Nextcloud davfs2 マウント設定（Ubuntu）

- <https://wiki.archlinux.jp/index.php/Davfs2>

## /etc/fstab

```fstab
https://nextcloud.example.com/remote.php/dav/files/myuser/ /mnt/nextcloud davfs rw,nofail,user,uid=myuser 0 0
```

## /etc/davfs2/secrets

```plain
https://nextcloud.example.com/remote.php/dav/files/myuser/ myuser APP_PASSWORD
```
