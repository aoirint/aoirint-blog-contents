---
title: 'docker-composeでBuildKitを使う'
date: '2021-11-08 07:30:00'
draft: false
category: Docker
tags:
  - Docker
  - 'Docker BuildKit'
  - 'Docker Compose'
---
# docker-composeでBuildKitを使う

## ビルドコマンド
- <https://stackoverflow.com/questions/58592259/how-do-you-enable-buildkit-with-docker-compose>

```shell
DOCKER_BUILDKIT=1 COMPOSE_DOCKER_CLI_BUILD=1 docker-compose build
```

## Dockerfile
- <https://www.docker.com/blog/introduction-to-heredocs-in-dockerfiles/>

```dockerfile
# syntax=docker/dockerfile:1.3-labs
```
