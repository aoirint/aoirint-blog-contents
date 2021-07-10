---
canonical_url: ./
title: 'GitHub Actions, DockerイメージをビルドしてDocker Hubにpushする（アクセストークン）'
date: '2021-07-10 10:10:00'
draft: false
category: GitHub
tags:
  - GitHub
  - GitHub Actions
  - 'CI CD'
  - Docker
---

# GitHub Actions, DockerイメージをビルドしてDocker Hubにpushする（アクセストークン）

Docker Hubにアクセストークンを追加する。

- [https://hub.docker.com/settings/security](https://hub.docker.com/settings/security)

リポジトリのSettingsから、Secretsを開き、New repository secretから
`DOCKER_USERNAME`、`DOCKER_PASSWORD`を追加する。

`DOCKER_USERNAME`はDocker Hub上のユーザ名、
`DOCKER_PASSWORD`はアクセストークンを設定する。

以下のファイルをリポジトリに追加する（ファイル名`docker.yml`は変更可）。
mainブランチにgit pushされたとき、Dockerイメージのビルドが走り、イメージがDocker Hubにdocker pushされる。

## .github/workflows/docker.yml
```
#!yaml
name: Push to Docker registry

on:
  push:
    branches:
      - main

jobs:
  push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Login to Docker Hub
        uses: actions-hub/docker/login@master
        env:
          DOCKER_USERNAME: ${{ secrets.DOCKER_USERNAME }}
          DOCKER_PASSWORD: ${{ secrets.DOCKER_PASSWORD }}
          DOCKER_REGISTRY_URL: docker.io

      - name: Build :latest
        if: success()
        run: docker build -t username/imagename:latest .

      - name: Deploy :latest
        if: success()
        uses: actions-hub/docker@master
        with:
          args: push username/imagename:latest
```
