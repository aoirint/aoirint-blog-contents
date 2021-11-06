---
title: GitLab CI, GitLab PagesにビルドしたHTMLを公開する
date: '2021-11-06 17:00:00'
draft: false
category: GitLab
tags:
  - GitLab
  - 'GitLab CI'
  - 'GitLab Pages'
  - 'CI CD'
---

# GitLab CI, GitLab PagesにビルドしたHTMLを公開する

- <https://docs.gitlab.com/ee/user/project/pages/>

リポジトリのGitLab Pages機能を有効化したあと、
GitLab CI上で`pages`というジョブに`public`というパスのArtifactがあるとき、
自動的に`pages:deploy`というジョブが実行され、GitLab Pagesへのデプロイが行われる。

リポジトリがプライベートリポジトリのとき、
デプロイされたGitLab Pagesは、GitLabアカウントで認証が行われる。

## .gitlab-ci.yml

- <https://docs.gitlab.com/ee/user/project/pages/getting_started/pages_from_scratch.html#specify-a-stage-to-deploy>

```
image: ruby:2.7

workflow:
  rules:
    - if: '$CI_COMMIT_BRANCH'

pages:
  stage: deploy
  script:
    - gem install bundler
    - bundle install
    - bundle exec jekyll build -d public
  artifacts:
    paths:
      - public
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
```
