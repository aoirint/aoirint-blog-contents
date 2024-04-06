---
title: GitLab CI, DockerイメージをビルドしてContainer Registryにpushする
date: '2021-05-29T02:00:00+09:00'
draft: false
channel: 技術ノート
category: GitLab
tags:
  - GitLab
  - 'GitLab CI'
  - 'CI CD'
  - Docker
---

# GitLab CI, DockerイメージをビルドしてContainer Registryにpushする

2023-05-18 追記：この記事には、[改訂版（2023年版）](/entry/2023/gitlab_ci_docker_build_2023/)があります。

---

## リポジトリ構造

- .gitlab-ci.yml
- app/
  - Dockerfile

## .gitlab-ci.yml

```yaml
stages:
  - build

build:
  stage: build
  image: docker:20.10
  services:
    - docker:dind
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
    - docker build ./app -t $CI_REGISTRY_IMAGE:latest
    - docker push $CI_REGISTRY_IMAGE:latest
```
