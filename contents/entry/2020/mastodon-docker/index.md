---
canonical_url: ./
title: Mastodonをdocker-composeで立てる（Ubuntu 18.04）
# og_image:
# twitter_card: summary_large_image
og_description: Mastodonをdocker-composeで立てる（Ubuntu 18.04）
date: '2020-12-06 11:00:00'
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
- [Mastodon documentation](https://docs.joinmastodon.org/)

内容はコミットID`44d5c6bc8ffd92cd201380dabe35748e50b6af68`、Mastodon Dockerイメージバージョン`v3.2.1`（Digest：`sha256:41cd5fb48d8b15ec806f08ab06fec98df33ec9b83a1f879e0fb30da9994018dc`）におけるもの。`docker-compose`の設定ファイルバージョンは`3`。

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

$ docker images tootsuite/mastodon --digests
REPOSITORY           TAG                 DIGEST                                                                    IMAGE ID            CREATED             SIZE
tootsuite/mastodon   v3.2.1              sha256:41cd5fb48d8b15ec806f08ab06fec98df33ec9b83a1f879e0fb30da9994018dc   37ca50fc92bd        6 weeks ago         1.86GB
```


今回はDocker Hub上のイメージを使用し、ローカルビルドをしない想定でいく（ごちゃごちゃするので）。
Mastodonを改造したい場合など、必要に応じて`github:tootsuite/mastodon`をFork/Cloneし、自分で/CIでビルドして信頼できるDockerレジストリに登録すればいいと思う。

Mastodonのリポジトリから`docker-compose.yml`、`.env.production.sample`をコピーしてくる。

環境変数設定ファイルである`.env.production.sample`を`.env.production`にリネームする。

ローカルビルドはしないので、web、streaming、sidekiqから`build: .`の行を削除する。
また、`image: tootsuite/mastodon:v3.2.1`のようにバージョンを固定しておく（実運用する場合は定期的に書き換えてバージョンアップ）。

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
`healthcheck`のところのユーザ名の書き換え、DB名の書き換えを忘れないように注意（`FATAL: role "postgres" does not exist`、`FATAL:  database "mastodon" does not exist`）。

```yaml
  db:
    restart: always
    image: postgres:9.6-alpine
    shm_size: 256mb
    networks:
      - internal_network
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "mastodon", "-D", "mastodon_production"]
    volumes:
      - ./postgres:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: mastodon
      POSTGRES_DB: mastodon_production
      POSTGRES_PASSWORD: YOUR_PASSWORD
```


Mastodonの環境変数設定ファイル`.env.production`の編集に移る。


Federationのセクションにいき、`LOCAL_DOMAIN`を編集する。
起動テスト目的なら適当なドメイン、またはngrokでHTTP 3000番（Mastodonのデフォルトポート）を開けておいてそのドメインを使うというのでいいと思う（そうした場合、実運用時は初期化した方がよさそうだが）。
ひとまずメールアドレス検証で送られるメールの確認リンクに使用されていた（適当なドメインを使用した場合はパスをコピーして使用すればよい）。

```
# Federation
# ----------
# This identifies your server and cannot be changed safely later
# ----------
LOCAL_DOMAIN=mstdn.example.com
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


HTTP 3000番から接続し、ブラウザ上でアカウントを作成する。
検証メールが届く。
その後アカウント`MY_USERNAME`を管理者に設定する。

```
docker-compose run --rm web bundle exec bin/tootctl accounts modify MY_USERNAME --role admin
```

またはコマンドで管理者アカウントを作成する。ランダムパスワードが生成され、標準出力に出力される。

```bash
sudo docker-compose run --rm web bundle exec bin/tootctl accounts create hoge --email hoge@example.com --confirmed --role admin
```

プロフィール画像アップロード時にエラーが出てしまった。
`./public/system`をマウント時にdockerが作成しているために`root`所有になっているのが原因。

```
Errno::EACCES (Permission denied @ dir_s_mkdir - /opt/mastodon/public/system/accounts)
```

`docker-compose run --rm web id -u`の出力は`991`だったので、ホスト側で`sudo chown -R 991:991 ./public`を実行して所有者を書き換えて解決した。

多人数が利用する OR 長期的に利用する予定で、VPSでホストするような場合、添付ファイルがVPSの容量を喰いつぶしてしまうことが想定されるので、そのような場合オブジェクトストレージを用意したい。


一人専用サーバの場合、`.env.production`に以下の設定を追加すると便利。`/`へのアクセスをユーザページにリダイレクトしてくれるようになり、新規登録を停止する。

```
SINGLE_USER_MODE=true
```


HTTPS化のためnginxによるリバースプロキシを設定する。
GitHub上にあった設定ファイルを参考にしている。
おそらく`X-Forwarded-Proto https`を設定しないと`https://localhost`にリダイレクトされるという事象が起こるので注意。
証明書の設定は書いていないが、`certbot`（Let's Encrypt）を使用する場合`sudo certbot --nginx`で自動挿入してくれる。
Mastodon側のポート番号はデフォルトのローカルループバックアドレスへのbindをそのまま使うことを想定。


```nginx
# https://github.com/tootsuite/mastodon/blob/master/nanobox/nginx-local.conf
# https://github.com/tootsuite/mastodon/blob/54192a9b6f8ee68114e3bc9ebf241099456e85f6/nanobox/nginx-local.conf

upstream rails {
  server 127.0.0.1:3000;
}

upstream node {
  server 127.0.0.1:4000;
}

map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}

server {
  server_name mstdn.example.com;

  keepalive_timeout 70;
  client_max_body_size 80M;

  add_header Strict-Transport-Security "max-age=31536000";

  location / {
    try_files $uri @rails;
  }

  location @rails {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header Proxy "";
    proxy_pass_header Server;

    proxy_pass http://rails;
    proxy_buffering off;
    proxy_redirect off;

    # WebSocket
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    
    tcp_nodelay on;
  }

  location /api/v1/streaming {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header Proxy "";
    proxy_pass_header Server;

    proxy_pass http://node;
    proxy_buffering off;
    proxy_redirect off;

    # WebSocket
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
    
    tcp_nodelay on;
  }

}
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
- [Mastodon 保守メモ - Qiita](https://qiita.com/kumasun/items/bf4997f181f893130041)
    - [勝手 Mastodon tootctl リファレンス - Qiita](https://qiita.com/kumasun/items/870769d7db4d95cde238)
    - 出回っている（日本語）情報はMastodonが流行った2017年前後のものが多いと思われるが、ここはかなり新しい情報が載っているようだ
