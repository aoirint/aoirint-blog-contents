---
title: 'Self-Hosting Overleaf Community Edition'
date: '2021-11-05 05:00:00'
draft: false
category: LaTeX
tags:
  - Overleaf
  - ShareLaTeX
---
# Self-Hosting Overleaf Community Edition

オンラインLaTeXエディタのShareLaTeXとOverleafは、Overleaf v2として2017年に統合され、OverleafはShareLaTeXのエディタを使うようになった。

- https://www.sharelatex.com/
- https://ja.overleaf.com/blog/sharelatex-joins-overleaf-2017-07-20

Overleaf（ShareLaTeX）は、`overleaf.com`で提供されているクラウド版と、セルフホスト可能なオープンソース版（Community Edition）が公開されている。

- <https://github.com/overleaf/overleaf>

この記事では、Overleaf Community Editionの公式Dockerイメージ（イメージ名は`sharelatex/sharelatex`）を使って、セルフホストする。

- <https://hub.docker.com/r/sharelatex/sharelatex/>

クラウド版とCommunity Editionの機能の違いは、以下を参照。

- <https://www.overleaf.com/for/enterprises/features>


## docker-compose.yml

- <https://github.com/overleaf/overleaf/blob/a752bbefdd7ef3316aaf0c34302f08e6024aaadb/docker-compose.yml>

```yaml
version: '3.9'
services:
  sharelatex:
    image: sharelatex/sharelatex:3
    restart: always
    depends_on:
      mongo:
        condition: service_healthy
      redis:
        condition: service_started
    ports:
      - "${SERVER_PORT}:80"
    volumes:
      - "${DATA_ROOT}:/var/lib/sharelatex"
      - "${TEXLIVE_ROOT}:/usr/local/texlive"
    environment:
      SHARELATEX_APP_NAME: Overleaf Community Edition
      SHARELATEX_MONGO_URL: mongodb://mongo/sharelatex
      SHARELATEX_REDIS_HOST: redis
      REDIS_HOST: redis

      ENABLED_LINKED_FILE_TYPES: 'project_file,project_output_file'

      # Enables Thumbnail generation using ImageMagick
      ENABLE_CONVERSIONS: 'true'

      # Disables email confirmation requirement
      EMAIL_CONFIRMATION_DISABLED: 'true'

      # temporary fix for LuaLaTex compiles
      # see https://github.com/overleaf/overleaf/issues/695
      TEXMFVAR: /var/lib/sharelatex/tmp/texmf-var

      # SHARELATEX_SITE_URL: http://sharelatex.mydomain.com
      # SHARELATEX_NAV_TITLE: Our ShareLaTeX Instance
      # SHARELATEX_HEADER_IMAGE_URL: http://somewhere.com/mylogo.png
      # SHARELATEX_ADMIN_EMAIL: support@it.com

      # SHARELATEX_LEFT_FOOTER: '[{"text": "Powered by <a href=\"https://www.sharelatex.com\">ShareLaTeX</a> 2016"},{"text": "Another page I want to link to can be found <a href=\"here\">here</a>"} ]'
      # SHARELATEX_RIGHT_FOOTER: '[{"text": "Hello I am on the Right"} ]'

      # SHARELATEX_EMAIL_FROM_ADDRESS: "team@sharelatex.com"

      # SHARELATEX_EMAIL_AWS_SES_ACCESS_KEY_ID:
      # SHARELATEX_EMAIL_AWS_SES_SECRET_KEY:

      # SHARELATEX_EMAIL_SMTP_HOST: smtp.mydomain.com
      # SHARELATEX_EMAIL_SMTP_PORT: 587
      # SHARELATEX_EMAIL_SMTP_SECURE: false
      # SHARELATEX_EMAIL_SMTP_USER:
      # SHARELATEX_EMAIL_SMTP_PASS:
      # SHARELATEX_EMAIL_SMTP_TLS_REJECT_UNAUTH: true
      # SHARELATEX_EMAIL_SMTP_IGNORE_TLS: false
      # SHARELATEX_EMAIL_SMTP_NAME: '127.0.0.1'
      # SHARELATEX_EMAIL_SMTP_LOGGER: true
      # SHARELATEX_CUSTOM_EMAIL_FOOTER: "This system is run by department x"

      ## Works with test LDAP server shown at bottom of docker compose
      # SHARELATEX_LDAP_URL: 'ldap://ldap:389'
      # SHARELATEX_LDAP_SEARCH_BASE: 'ou=people,dc=planetexpress,dc=com'
      # SHARELATEX_LDAP_SEARCH_FILTER: '(uid={{username}})'
      # SHARELATEX_LDAP_BIND_DN: 'cn=admin,dc=planetexpress,dc=com'
      # SHARELATEX_LDAP_BIND_CREDENTIALS: 'GoodNewsEveryone'
      # SHARELATEX_LDAP_EMAIL_ATT: 'mail'
      # SHARELATEX_LDAP_NAME_ATT: 'cn'
      # SHARELATEX_LDAP_LAST_NAME_ATT: 'sn'
      # SHARELATEX_LDAP_UPDATE_USER_DETAILS_ON_LOGIN: 'true'

      # SHARELATEX_TEMPLATES_USER_ID: "578773160210479700917ee5"
      # SHARELATEX_NEW_PROJECT_TEMPLATE_LINKS: '[ {"name":"All Templates","url":"/templates/all"}]'

      # SHARELATEX_PROXY_LEARN: "true"

  mongo:
    image: mongo:4.0
    restart: always
    volumes:
      - "${MONGO_ROOT}:/data/db"
    healthcheck:
      test: echo 'db.stats().ok' | mongo localhost:27017/test --quiet
      interval: 10s
      timeout: 10s
      retries: 5

  redis:
    image: redis:5
    restart: always
    volumes:
      - "${REDIS_ROOT}:/data"

  # ldap:
  #  image: rroemhild/test-openldap
  #  restart: always
```

## .env

```env
SERVER_PORT=127.0.0.1:8000
DATA_ROOT=./data/sharelatex
TEXLIVE_ROOT=./data/texlive
MONGO_ROOT=./data/mongo
REDIS_ROOT=./data/redis
```

## 設定

- <https://github.com/overleaf/overleaf/wiki/Quick-Start-Guide>

## TeXLiveのフルインストール

TeXLiveのフルバージョンは巨大なため、Dockerイメージには最小構成のみが含まれている。
フルバージョンを使うには、インストールコマンドを実行する必要がある。

また、コンテナが削除されるとインストールしたTeXLiveが消え、再インストールが必要になるため、マウントして永続化する。
マウントすると、もともとイメージにあったディレクトリが上書きされるので、あらかじめイメージ内のTeXLiveをマウントするディレクトリにコピーしておく。

```shell
docker pull sharelatex/sharelatex:3
docker run -d --name sharelatex sharelatex/sharelatex:3
docker cp sharelatex:/usr/local/texlive ./data/
docker rm -f sharelatex

docker-compose pull
docker-compose up -d

docker-compose exec sharelatex tlmgr option repository https://mirror.ctan.org/systems/texlive/tlnet
docker-compose exec sharelatex tlmgr update --self
docker-compose exec sharelatex tlmgr install scheme-full
```

- http://www.fugenji.org/~thomas/texlive-guide/tlmgr.html

なお、公式Wikiでは、ローカル用のDockerイメージを作る方法が案内されている。

- <https://github.com/overleaf/overleaf/wiki/Quick-Start-Guide#latex-environment>

## ユーザの登録

- <https://github.com/overleaf/overleaf/wiki/Quick-Start-Guide#creating-and-managing-users>

初期状態では、`/launchpad`にアクセスすることで管理ユーザを作成できる（コマンドで作成も可）。

一般ユーザの登録には、`/admin/register`にアクセスする。
