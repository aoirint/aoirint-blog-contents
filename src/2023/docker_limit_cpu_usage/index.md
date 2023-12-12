---
title: 'DockerコンテナのCPU使用量を制限する'
date: '2023-12-09T14:00:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Docker
tags:
  - Docker
  - 'Docker Compose'
---
# DockerコンテナのCPU使用量を制限する

- Docker Engine 24.0
- Docker Compose 2.21

## Docker

```shell
# CPU 1個
sudo docker update --cpus "1" "$CONTAINER_ID"

# CPU 0.01個（最小）
sudo docker update --cpus "0.01" "$CONTAINER_ID"

# 制限を解除
sudo docker update --cpus "0" "$CONTAINER_ID"
```

## Docker Compose

```yaml
app:
  image: hello-world
  deploy:
    resources:
      limits:
        cpus: '0.1'
```

## 参考

- [Update the limitation of memory/CPU for existing container in docker - Stack Overflow](https://stackoverflow.com/questions/34654697/update-the-limitation-of-memory-cpu-for-existing-container-in-docker)
- [Runtime options with Memory, CPUs, and GPUs | Docker Docs](https://docs.docker.com/config/containers/resource_constraints/#configure-the-default-cfs-scheduler)
- [Compose Deploy Specification | Docker Docs](https://docs.docker.com/compose/compose-file/deploy/#resources)
