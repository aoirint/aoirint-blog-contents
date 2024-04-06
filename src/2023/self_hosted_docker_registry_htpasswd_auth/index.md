---
title: 'Docker Registryをホストしてhtpasswdで認証する'
date: '2023-12-09T14:50:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Docker
tags:
  - 'Docker Registry'
  - Docker
  - 'Docker Compose'
---
# Docker Registryをホストしてhtpasswdで認証する

- [registry - Official Image | Docker Hub](https://hub.docker.com/_/registry/)
- [Native basic auth - Restricting access - Deploy a registry server | CNCF Distribution](https://distribution.github.io/distribution/about/deploying/#native-basic-auth)

## htpasswdファイルの作成

htpasswdファイルを作成します。
registryイメージ（Distribution Registry）はbcrypt形式のパスワードのみサポートしているため、パスワードをbcrypt形式でハッシュ化する必要があります。

```shell
sudo apt install apache2-utils

mkdir auth
cd auth

htpasswd -cB htpasswd myuser
```

## 永続化ディレクトリの作成

```shell
mkdir data
```

## docker-compose.ymlファイルの作成

```yaml
services:
  registry:
    image: registry:2
    restart: always
    environment:
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
    # ports:
    #   - "0.0.0.0:5000:5000"
    volumes:
      - ./data:/var/lib/registry
      - ./auth:/auth
```

## コンテナの実行

```shell
sudo docker compose up -d
```

コンテナのTCP 5000番ポートでDocker Registry HTTP APIがリッスンします。
このポート宛にCloudflaredやリバースプロキシを設定して、`https://docker.example.com`のようにサービスを公開します。

設定後、`docker`コマンドから以下のように利用できます（プッシュ・プルいずれも認証が必要）。

```shell
sudo docker login -u myuser docker.example.com

sudo docker build -t docker.example.com/myuser/myimage:0.1.0 .
sudo docker push docker.example.com/myuser/myimage:0.1.0

sudo docker pull docker.example.com/myuser/myimage:0.1.0
```
