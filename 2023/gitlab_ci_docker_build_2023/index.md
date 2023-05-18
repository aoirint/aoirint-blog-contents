---
title: 'GitLab CI, DockerイメージをビルドしてContainer Registryにpushする（2023年版）'
date: '2023-05-18T11:00:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: GitLab
tags:
  - GitLab
  - 'GitLab CI'
  - 'CI CD'
  - Docker
---
# GitLab CI, DockerイメージをビルドしてContainer Registryにpushする（2023年版）

[前回の記事（2021年版）](/entry/2021/gitlab_ci_docker_registry/)から、以下の内容でアップデートしました。

- Docker Engine 23.0
- BuildKit
- レイヤーキャッシュ（Registry cache）
- タグによるバージョン付け

## 注意：Self Hosted GitLab RunnerでのDockerイメージビルドは推奨しません

DinDでのビルドのため、ホストOSのroot権限が取得可能な、コンテナの特権実行（`--privileged`）、またはDockerソケットのマウント（DooD）が要求されます。

Shared Runnerは使い捨てのGCPインスタンスで提供されるため、コンテナブレイクアウト等によりホストのroot権限が取得されても、
Dockerエンジンのホストである仮想マシンごと破棄されますが、そのような工夫をしていないRunner（VPSやベアメタル）では、
CIジョブの実行により、ホストのroot権限で悪意ある操作が実行され、また、その影響が持続する危険性があります。

Dockerデーモンを必要としない、代替ソフトウェアによるDockerイメージビルドを検討してください。

## リポジトリ構造

- .gitlab-ci.yml
- Dockerfile

## .gitlab-ci.yml

```yaml
# License: CC0-1.0
stages:
  - build

build:
  stage: build
  image: docker:23.0
  services:
    - docker:dind
  rules:
    # Release
    - if: $CI_COMMIT_TAG
      variables:
        DOCKER_IMAGE_NAME_AND_TAG: "${CI_REGISTRY_IMAGE}:${CI_COMMIT_TAG}"
        DOCKER_CACHE_FROM: "type=registry,ref=${CI_REGISTRY_IMAGE}:latest-buildcache"
        DOCKER_CACHE_TO: ""
    # Default branch
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      variables:
        DOCKER_IMAGE_NAME_AND_TAG: "${CI_REGISTRY_IMAGE}:latest"
        DOCKER_CACHE_FROM: "type=registry,ref=${CI_REGISTRY_IMAGE}:latest-buildcache"
        DOCKER_CACHE_TO: "type=registry,ref=${CI_REGISTRY_IMAGE}:latest-buildcache,mode=max"
  script:
    - apk add --no-cache git
    - docker buildx create --use
    - docker login -u "${CI_REGISTRY_USER}" -p "${CI_REGISTRY_PASSWORD}" "${CI_REGISTRY}"
    - >
      docker buildx build .
      -t "${DOCKER_IMAGE_NAME_AND_TAG}"
      --cache-from "${DOCKER_CACHE_FROM}"
      --cache-to "${DOCKER_CACHE_TO}"
      --push
```

- ※ docker buildx buildのコマンドは複数行になっていますが、2行目以降を1行目と異なるインデント数にしないでください。2行目以降が別のコマンド扱いになり、動作しなくなります。
