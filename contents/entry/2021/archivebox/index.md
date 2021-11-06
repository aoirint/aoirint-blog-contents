---
title: Archivebox
date: '2021-11-06 23:00:00'
draft: false
category: WebClip
tags:
  - WebClip
  - Archivebox
---
# Archivebox

- <https://github.com/ArchiveBox/ArchiveBox>
- <https://hub.docker.com/r/archivebox/archivebox>

```shell
curl -O 'https://raw.githubusercontent.com/ArchiveBox/ArchiveBox/master/docker-compose.yml'
```

## docker-compose.yml

```yaml
# Usage:
#     docker-compose run archivebox init --setup
#     docker-compose up
#     echo "https://example.com" | docker-compose run archivebox archivebox add
#     docker-compose run archivebox add --depth=1 https://example.com/some/feed.rss
#     docker-compose run archivebox config --set PUBLIC_INDEX=True
#     docker-compose run archivebox help
# Documentation:
#     https://github.com/ArchiveBox/ArchiveBox/wiki/Docker#docker-compose

version: "3.9"

services:
    archivebox:
        image: archivebox/archivebox:sha-f809e3b
        command: server --quick-init 0.0.0.0:8000
        ports:
          - "${SERVER_PORT:-8000}:8000"
        environment:
            ALLOWED_HOSTS: "*"
            MEDIA_MAX_SIZE: 2g
            # SEARCH_BACKEND_ENGINE: sonic     # uncomment these if you enable sonic below
            # SEARCH_BACKEND_HOST_NAME: sonic
            # SEARCH_BACKEND_PASSWORD: SecretPassword
        volumes:
            - "${DATA_ROOT:-./data}:/data"

    # To run the Sonic full-text search backend, first download the config file to sonic.cfg
    # curl -O https://raw.githubusercontent.com/ArchiveBox/ArchiveBox/master/etc/sonic.cfg
    # after starting, backfill any existing Snapshots into the index: docker-compose run archivebox update --index-only
    # sonic:
    #    image: valeriansaliou/sonic:v1.3.0
    #    environment:
    #        SEARCH_BACKEND_PASSWORD: SecretPassword
    #    volumes:
    #        - ./sonic.cfg:/etc/sonic.cfg:ro
    #        - ./data/sonic:/var/lib/sonic/store


    ### Optional Addons: tweak these examples as needed for your specific use case

    # Example: Run scheduled imports in a docker instead of using cron on the
    # host machine, add tasks and see more info with archivebox schedule --help
    # scheduler:
    #    image: archivebox/archivebox:latest
    #    command: schedule --foreground --every=day --depth=1 'https://getpocket.com/users/USERNAME/feed/all'
    #    environment:
    #        USE_COLOR: True
    #        SHOW_PROGRESS: False
    #    volumes:
    #        - ./data:/dat
```

## .env

```env
SERVER_PORT=127.0.0.1:8000
DATA_ROOT=./data
```

## 初期設定

管理ユーザの作成、プライベートサーバ化、再起動。

```shell
docker-compose exec -u "archivebox" archivebox archivebox manage createsuperuser

docker-compose exec -u "archivebox" archivebox archivebox config --set PUBLIC_INDEX=False
docker-compose exec -u "archivebox" archivebox archivebox config --set PUBLIC_SNAPSHOTS=False
docker-compose exec -u "archivebox" archivebox archivebox config --set PUBLIC_ADD_VIEW=False

docker-compose up -d --force-recreate
```

`archive.org`への保存を試みるらしいので、無効化したい場合は以下の設定が必要。

- <https://github.com/ArchiveBox/ArchiveBox#archiving-private-content>

```shell
docker-compose exec -u "archivebox" archivebox archivebox config --set SAVE_ARCHIVE_DOT_ORG=False
```
