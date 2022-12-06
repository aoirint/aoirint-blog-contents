---
title: WebDAV in Docker
date: '2021-08-22T11:15:00+09:00'
draft: false
channel: 技術ノート
category: Network
tags:
  - Network
  - WebDAV
  - Docker
---
# WebDAV in Docker

- [https://github.com/aoirint/webdav-docker](https://github.com/aoirint/webdav-docker)
- [https://hub.docker.com/r/aoirint/webdav](https://hub.docker.com/r/aoirint/webdav)

以下のリポジトリをforkし、Windows 10のExplorerクライアントに対応させたDockerイメージ。
Apache Web ServerのDAV機能でWebDAVサーバを立てる。

- [https://github.com/BytemarkHosting/docker-webdav](https://github.com/BytemarkHosting/docker-webdav)

## docker-compose.yml

```yaml
version: '3.9'
services:
  webdav:
    image: aoirint/webdav:2.4-20210822c
    restart: always
    ports:
      - '${DAV_PORT:-127.0.0.1:8000}:80'
    environment:
      LOCATION: /webdav
      ANONYMOUS_METHODS: OPTIONS
      AUTH_TYPE: Basic
      USERNAME: ${DAV_USERNAME:-user}
      PASSWORD: ${DAV_PASSWORD:-password}
      # SKIP_CHOWN: 1
    volumes:
      - ./dav:/var/lib/dav
```

以上の設定で、`dav://127.0.0.1:8000/webdav`にWebDAVサーバが立つ。

データは`./dav/data`に格納される。


## Optional: /etc/fstab

シンボリックリンクは動作しないので、`bindfs`を使う。

```shell
sudo apt install bindfs fuse-utils
```

```fstab
/src/path /dest/dav/data/path fuse.bindfs rw,user,uid=YOURUSER 0 0
```

- [https://www.netfort.gr.jp/~tosihisa/notebook/doku.php/bindfs](https://www.netfort.gr.jp/~tosihisa/notebook/doku.php/bindfs)
