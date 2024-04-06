---
# moved from https://aoirint.hatenablog.com/entry/2019/11/27/020647
title: Raspberry Pi 3B+（Raspbian）でNextcloud（Docker）を動かす（MySQL）
date: '2019-11-27T02:06:47+09:00'
draft: false
channel: 技術ノート
category: Docker
tags:
- Docker
- CloudStorage
---
# Raspberry Pi 3B+（Raspbian）でNextcloud（Docker）を動かす（MySQL）

※ Dockerは入ってるものとします。あとスワップ領域を用意しておいたほうがいいかな

```shell
docker run --name ncdb --restart unless-stopped -v NEXTCLOUD_DIR/db:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=MY_ROOT_PASSWORD -e MYSQL_DATABASE=nextcloud -e MYSQL_USER=nextcloud -e MYSQL_PASSWORD=MY_PASSWORD hypriot/rpi-mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci

docker run --name ncapp --restart unless-stopped --link ncdb -p MY_PORT:80 -v NEXTCLOUD_DIR/www:/var/www/html -e MYSQL_DATABASE=nextcloud -e MYSQL_USER=nextcloud -e MYSQL_PASSWORD=MY_PASSWORD -e MYSQL_HOST=ncdb:3306 -e NEXTCLOUD_TRUSTED_DOMAINS="MY_DOMAIN" nextcloud
```

- `NEXTCLOUD_DIR/www/config/config.php`
  - `overwriteprotocol: 'https'`
  - `overwritewebroot: ''`
  - `overwrite.cli.url: 'https://MY_DOMAIN'`
  - この設定をしないとhttps環境ではクライアントからログイン（Grant access）できない（webrootはいらないかな）
