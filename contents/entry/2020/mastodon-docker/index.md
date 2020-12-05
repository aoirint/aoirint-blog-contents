---
canonical_url: ./
title: Mastodonをdocker-composeで立てる（Ubuntu 18.04）
# og_image:
# twitter_card: summary_large_image
og_description: Mastodonをdocker-composeで立てる（Ubuntu 18.04）
date: '2020-12-06 08:40:00'
draft: false
category: 記事
tags:
  - Mastodon
  - docker-compose
  - Docker
  - Ubuntu
---

# Mastodonをdocker-composeで立てる（Ubuntu 18.04）
- [tootsuite/mastodon: Your self-hosted, globally interconnected microblogging community](https://github.com/tootsuite/mastodon)

内容はコミットID`44d5c6bc8ffd92cd201380dabe35748e50b6af68`におけるもの。`docker-compose`の設定ファイルバージョンは`3`。

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 18.04.5 LTS
Release:	18.04
Codename:	bionic

$ uname -r
5.4.0-56-generic

$ docker -v
Docker version 19.03.14, build 5eb3275d40

$ docker-compose -v
docker-compose version 1.27.1, build 509cfb99
```


今回はDocker Hub上のイメージを使用し、ローカルビルドをしない想定でいく（ごちゃごちゃするので）。
Mastodonを改造したい場合など、必要に応じて`github:tootsuite/mastodon`をFork/Cloneし、自分で/CIでビルドして信頼できるDockerレジストリに登録すればいいと思う。

Mastodonのリポジトリから`docker-compose.yml`、`.env.production.sample`をコピーしてくる。

環境変数設定ファイルである`.env.production.sample`を`.env.production`にリネームする。

ローカルビルドはしないので、web、streaming、sidekiqから`build: .`の行を削除しておく。

DB、Redis、Mastodon各サービスの最新のDockerイメージを取得する。

```bash
docker-compose pull
```


DB（PostgreSQL）のパスワードを生成する。

```bash
# 適当な長さで
# rake secretを使ってもいいのだろうか?
pwgen 32
```

`docker-compose.yml`中のDB部分にDB名・ユーザ名・パスワードを設定する
（`docker-compose.yml`に直接書きたくない場合は`.env.db`などを作成、`env_file: `以下にファイルパスを設定する。または`docker-compose.override.yml`を作成する）。
`healthcheck`のところのユーザ名を書き換え忘れないように注意（`FATAL: role "postgres" does not exist`）。

```yaml
  db:
    restart: always
    image: postgres:9.6-alpine
    shm_size: 256mb
    networks:
      - internal_network
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "mastodon"]
    volumes:
      - ./postgres:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: mastodon
      POSTGRES_DB: mastodon_production
      POSTGRES_PASSWORD: YOUR_PASSWORD
```


Mastodonの環境変数設定ファイル`.env.production`の編集に移る。


Federationのセクションにいき、`LOCAL_DOMAIN`を編集する。
起動テスト目的なら適当なドメインでいいと思うが、ngrokでHTTP 3000番（Mastodonのデフォルトポート）を開けておいてそのドメインを使ってもよいと思う。
ひとまずメールアドレス検証で送られるメールの確認リンクに使用されていた（適当なドメインを使用した場合はパスをコピーして使用すればよい）。

```
# Federation
# ----------
# This identifies your server and cannot be changed safely later
# ----------
LOCAL_DOMAIN=hogehoge.ngrok.io
```


Redisのセクションにいき、`REDIS_HOST`を編集する。
docker-composeが作成するネットワークを使用するので、ホスト名`redis`（サービス名）で接続できる。

```env
# Redis
# -----
REDIS_HOST=redis
REDIS_PORT=6379
```


PostgreSQLのセクションにいき、`DB_HOST`、`DB_PASS`を編集する。
docker-composeが作成するネットワークを使用するので、ホスト名`db`（サービス名）で接続できる。

```env
# PostgreSQL
# ----------
DB_HOST=db
DB_USER=mastodon
DB_NAME=mastodon_production
DB_PASS=YOUR_PASSWORD
DB_PORT=5432
```

ひとまず全文検索エンジンElasticSearchは無効化しておく。

```env
# ElasticSearch (optional)
# ------------------------
ES_ENABLED=false
```


セッション用と二要素認証用の2つのシークレットをDocker Hub上のMastodonイメージ（`docker.io/tootsuite/mastodon`）内の`Rakefile`を使って生成する。
標準出力にランダム文字列が吐き出されるのでコピーする。

```bash
# for SECRET_KEY_BASE
docker run --rm tootsuite/mastodon bundle exec rake secret

# for OTP_SECRET
docker run --rm tootsuite/mastodon bundle exec rake secret
```


Web Pushの公開鍵・秘密鍵を生成する（環境変数を設定しないとエラー）。標準出力に.envの形式で吐き出されるのでコピーする。

```bash
docker run --rm --env-file ./.env.production tootsuite/mastodon bundle exec rake mastodon:webpush:generate_vapid_key
```


メールアドレス検証・通知などに使うメールサーバ（SMTPサーバ）を設定する。
今回は面倒なので自分のGoogleアカウントを使用する。
Googleアカウントの二要素認証が有効になっていることを確認し、
Googleアカウント設定からメールに使用するアプリパスワードを生成する。

```env
# Sending mail
# ------------
SMTP_SERVER=smtp.gmail.com
SMTP_PORT=587
SMTP_LOGIN=YOUR_NAME@gmail.com
SMTP_PASSWORD=YOUR_APP_PASSWORD
SMTP_FROM_ADDRESS=YOUR_NAME@gmail.com
```


オブジェクトストレージ接続機能はひとまず無効化しておく。

```env
# File storage (optional)
# -----------------------
S3_ENABLED=false
```

設定ファイルの編集は以上。


DBの初期化と静的ファイル生成。

```bash
docker-compose run --rm web rails db:migrate

docker-compose run --rm web rails assets:precompile
```


docker-compose run --rmでweb以外のコンテナが止まらないので一度すべてのコンテナを停止・削除する。
これにより永続化が正しく動作していることを確認できる。

```bash
docker-compose down
```


本起動。
`restart: always`が設定されているため`docker-compose down`しない限りはホスト再起動時も自動で起動する。

```bash
docker-compose up -d
```

途中で操作間違えたり、DB初期化中にkillしたりして失敗したときはコンテナ・マウントディレクトリを削除して初期化。
```bash
docker-compose down
sudo rm -rf postgres/ public/ redis/
```


## 参考
- [tootsuite/mastodon: Your self-hosted, globally interconnected microblogging community](https://github.com/tootsuite/mastodon)
- [Configuring your environment - Mastodon documentation](https://docs.joinmastodon.org/admin/config/)
  - Docker用のドキュメントはないのか?
- [tootsuite/mastodon - Docker Hub](https://hub.docker.com/r/tootsuite/mastodon)
- [postgres - Docker Hub](https://hub.docker.com/_/postgres)
- [redis - Docker Hub](https://hub.docker.com/_/redis)
- docker-compose 2 時代の記事
  - [Docker応用チュートリアル：Mastodon - Qiita](https://qiita.com/zembutsu/items/f1f1ede26102ba27fce2)
  - [Dockerで雑にMastodonを起動する方法 - Qiita](https://qiita.com/zembutsu/items/fd52a504321dd5d6f0b8)
- [Rails5をproduction（本番環境）で起動する時に嵌ったこと - Qiita](https://qiita.com/qqhann/items/7cd01f4b5cff4a31e053)
