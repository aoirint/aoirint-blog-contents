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

## 注意：Self Hosted GitLab RunnerでのDockerデーモンを使ったイメージビルドは推奨しません

DinDでのビルドのため、ホストOSのroot権限が取得可能な、コンテナの特権実行（`--privileged`）、またはDockerソケットのマウント（DooD）が要求されます。

GitLab.comのShared Runnerは使い捨てのGCPインスタンスで提供されるため、コンテナブレイクアウト等によりホストのroot権限が取得されても、
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

### `docker buildx create --use`

`docker buildx build`実行時に、以下のようなエラーが出るため追加しています。

```
ERROR: cache export feature is currently not supported for docker driver. Please switch to a different driver (eg. "docker buildx create --use")
```

## 関連トピック：SLSA

SLSA（Supply-chain Levels for Software Artifacts、サルサ）は、ソフトウェアのサプライチェーンを検証可能にしようという仕組みです。
米国の大統領令EO14028、EUのサイバーレジリエンス法（CRA）などの規制に関連する可能性があります。

Dockerでは、`--provenance`オプションが導入され、DockerイメージにSLSA関連のメタデータを付加することができます。

- [docker/build-push-action v3.3.0で導入されたprovenanceオプションにまつわる問題 - chroju.dev/blog](https://chroju.dev/blog/docker_buildx_slsa_provenance)
- [【Infostand海外ITトピックス】オープンソース業界に広がる懸念　欧州で導入予定のサイバーレジリエンス法 - クラウド Watch](https://cloud.watch.impress.co.jp/docs/column/infostand/1497776.html)
- [米国大統領令とEUのCRAが示すソフトウェアサプライチェーンセキュリティとは：ソフトウェアサプライチェーンの守り方（1）（1/2 ページ） - MONOist](https://monoist.itmedia.co.jp/mn/articles/2212/06/news004.html)
- [SLSA • Supply-chain Levels for Software Artifacts](https://slsa.dev/)
- [SLSAとは【用語集詳細】](https://www.sompocybersecurity.com/column/glossary/slsa)
- [aoirint🎐: "👀 / docker/build-push-action …" - mstdn.aoirint.com](https://mstdn.aoirint.com/@aoirint/110224046831353435)

## 参考

- [Release CI/CD examples | GitLab](https://docs.gitlab.com/ee/user/project/releases/release_cicd_examples.html)
- [YAMLで複数行テキストを書きたい時のあれこれ - Qiita](https://qiita.com/jerrywdlee/items/d5d31c10617ec7342d56)
- [GitLab CIでDockerイメージをビルドする - Qiita](https://qiita.com/MH35JP/items/ba2147b8d153a1500899#buildx%E3%82%92%E4%BD%BF%E3%81%84%E3%81%9F%E3%81%84)
- [Registry cache | Docker Documentation](https://docs.docker.com/build/cache/backends/registry/)
- [if statement - How to use if-else condition on gitlabci - Stack Overflow](https://stackoverflow.com/questions/54761464/how-to-use-if-else-condition-on-gitlabci)
- [GitLab CI/CD variables | GitLab](https://docs.gitlab.com/ee/ci/variables/)
- [Choose when to run jobs | GitLab](https://docs.gitlab.com/ee/ci/jobs/job_control.html)

## その他リンク

- [実験用リポジトリ（GitLab.com）](https://gitlab.com/aoirint/docker_ci_example)
