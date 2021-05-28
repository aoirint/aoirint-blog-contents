---
canonical_url: ./
title: GitLab CIでDockerイメージをビルドしてContainer Registryにpushする
date: '2021-05-29 02:00:00'
draft: false
category: GitLab
tags:
  - GitLab
  - 'GitLab Actions'
  - 'CI CD'
  - Docker
---

# GitLab CIでDockerイメージをビルドしてContainer Registryにpushする

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
