---
title: 'Dockerコンテナ内の名前解決をプライベートIPアドレスにする'
date: '2023-12-09T16:50:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Docker
tags:
  - Docker
  - 'Docker Compose'
  - Network
---
# Dockerコンテナ内の名前解決をプライベートIPアドレスにする

- Docker Engine 24.0
- Docker Compose 2.21

プライベートDNSを運用しているネットワークで、
Dockerコンテナからネットワーク内の別のホストに通信したいとき、
デフォルトではGoogle Public DNS`8.8.8.8`でホスト名がグローバルIPアドレスに解決されるため、
ISPやCloudflare Tunnelなどの外部を経由して通信することになり、非効率で危険な通信経路になります。

また、IPアドレスによるアクセス制限を設けている場合、
プライベートIPアドレスがソースとなる通信になるように通信経路を制御したい場合があります。

以下の設定により、コンテナ内で`example.com`が`192.168.0.50`に解決されるようになります。

## Docker

```shell
sudo docker run --rm --add-host "example.com:192.168.0.50" hello-world
```

- [Managing /etc/hosts - Docker run reference | Docker Docs](https://docs.docker.com/engine/reference/run/#managing-etchosts)

## Docker Compose

```yaml
services:
  app:
    image: hello-world
    extra_hosts:
      - "example.com:192.168.0.50"
```

- [extra_hosts - Services top-level element | Docker Docs](https://docs.docker.com/compose/compose-file/05-services/#extra_hosts)
