---
title: WebDAV in Docker
date: '2021-08-22 11:15:00'
draft: false
category: Network
tags:
  - Network
  - WebDAV
---
# WebDAV in Docker

- <https://github.com/aoirint/webdav-docker>
- <https://hub.docker.com/r/aoirint/webdav>

以下のリポジトリをWindows 10のExplorerクライアントに対応させたDockerイメージ。
Apache Web ServerのDAV機能でWebDAVサーバを立てる。

- <https://github.com/BytemarkHosting/docker-webdav>

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
