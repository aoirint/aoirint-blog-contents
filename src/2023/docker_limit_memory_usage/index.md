---
title: 'Dockerコンテナのメモリ使用量を制限する'
date: '2023-12-09T14:10:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Docker
tags:
  - Docker
  - 'Docker Compose'
---
# Dockerコンテナのメモリ使用量を制限する

- Docker Engine 24.0
- Docker Compose 2.21

## Docker

```shell
# 100 MB
sudo docker update --memory "100m" "$CONTAINER_ID"

# 1 GB
sudo docker update --memory "1g" "$CONTAINER_ID"

# 制限を解除
sudo docker update --memory "0" "$CONTAINER_ID"
```

## Docker Compose

```yaml
app:
  image: hello-world
  deploy:
    resources:
      limits:
        memory: '1g'
```

## 参考

- [Update the limitation of memory/CPU for existing container in docker - Stack Overflow](https://stackoverflow.com/questions/34654697/update-the-limitation-of-memory-cpu-for-existing-container-in-docker)
- [Runtime options with Memory, CPUs, and GPUs | Docker Docs](https://docs.docker.com/config/containers/resource_constraints/#configure-the-default-cfs-scheduler)
- [Compose Deploy Specification | Docker Docs](https://docs.docker.com/compose/compose-file/deploy/#resources)
