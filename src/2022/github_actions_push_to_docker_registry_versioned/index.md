---
title: GitHub ActionsでDocker RegistryにDockerイメージをpushする（latestタグ、GitHub Release連携でバージョン付け）
date: '2022-06-24T09:10:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: GitHub
tags:
  - GitHub
  - Docker
---
# GitHub ActionsでDocker RegistryにDockerイメージをpushする（latestタグ、GitHub Release連携でバージョン付け）

- [https://blog.aoirint.com/entry/2021/github_actions_docker_io_token/](https://blog.aoirint.com/entry/2021/github_actions_docker_io_token/)

上の記事のWorkflowテンプレートをちょっと改良した。

- Docker Hub以外のDockerレジストリに対応
- GitHub Release作成時にリリースタグをイメージタグにしてpush

## レジストリURL

- Docker Hub: `docker.io`
- GitHub Container Registry: `ghcr.io`
- GitLab Container Registry: `registry.gitlab.com`

## GitHub Secrets

- DOCKER_USERNAME
- DOCKER_TOKEN

## GitHub Workflow .github/workflows/docker.yml

- [https://github.com/docker/login-action](https://github.com/docker/login-action)
- [https://github.com/docker/build-push-action](https://github.com/docker/build-push-action)

```yaml
name: Push to Docker registry

on:
  push:
    branches:
      - main
  release:
    types:
      - created

env:
  IMAGE_NAME: docker.io/username/imagename
  IMAGE_TAG: ${{ github.event.release.tag_name != '' && github.event.release.tag_name || 'latest' }}

jobs:
  push:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Docker Buildx
        id: buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Docker Registry
        uses: docker/login-action@v2
        with:
          registry: docker.io
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}

      - name: Build and Deploy Docker image
        uses: docker/build-push-action@v3
        env:
          IMAGE_NAME_AND_TAG: ${{ format('{0}:{1}', env.IMAGE_NAME, env.IMAGE_TAG) }}
        with:
          context: .
          builder: ${{ steps.buildx.outputs.name }}
          file: ./Dockerfile
          push: true
          tags: ${{ env.IMAGE_NAME_AND_TAG }}
          cache-from: type=registry,ref=${{ env.IMAGE_NAME_AND_TAG }}-buildcache
          cache-to: type=registry,ref=${{ env.IMAGE_NAME_AND_TAG }}-buildcache,mode=max
```
