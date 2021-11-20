---
title: Dockerでプロキシ・バーチャルホストに対応したSSH Serverを立てる
# og_image:
# twitter_card: summary_large_image
og_description: Dockerでプロキシ・バーチャルホストに対応したSSH Serverを立てる
date: '2020-10-15 23:00:00'
draft: true
category: リモート環境
tags:
  - Docker
  - ssh
---
# Dockerでプロキシ・バーチャルホストに対応したSSH Serverを立てる

本記事は、ホストのSSHサーバと独立した仮想SSHサーバを立てることを目標とする。

クライアントが、限られたサーバ側ポート（22、80、443）の通信のみが通過できるネットワークに接続しているとき、
ホストで動かしているサービスと同じサービスを新たに立てようとすると
ポート番号が重複してしまい、接続可能な状態にすることができない。

HTTPではWebサーバのバーチャルホスト機能を使って
ドメインを切り分けるなどしてポート重複を回避できるが、
SSHではプロトコル上、HTTPのようにホスト名を使ってバーチャルホストを実現することができない。

本記事では、以下のようにリバースプロキシ・HTTPプロキシを
SSHサーバの前段階に設けることでHTTPと同様なアクセス振り分けを可能にする。

```
クライアント
→クライアント側ネットワーク（:80 or :443のみ通過可）
→リバースプロキシ（Webサーバ） :80 or :443
→HTTPプロキシ internal port
→SSHサーバ :22
```

## SSHサーバを立てる
SSHサーバを立てる。Dockerfileは[maltyxx/sshd](https://github.com/maltyxx/docker-sshd)を使う。

### docker-compose.yml

debianのリポジトリとリンクして自動ビルドされるようなことが書いてあるがなんか止まってて
ちょっと古いので、ローカルでビルドすることにする。
`docker-compose.yml`を書いたら`docker-compose build`を実行してビルドする。

```
Commit ID: 24a1f32f6060054c64e294d3b074885c856b29d2
```


```yaml
version: '3.8'
services:
  sshd:
    image: maltyxx/sshd
    build: https://github.com/maltyxx/docker-sshd.git
    restart: always
    command: 'myuser::1000'
    ports:
      - '2222:22'
    volumes:
      - './YOUR_PUBLIC_KEY.pub:/home/myuser/.ssh/keys/YOUR_PUBLIC_KEY.pub:ro'
```

#### 説明
コマンド部分にユーザ情報を書くと自動でコンテナ内ユーザが作成される（README参照、複数ユーザも可）。
ここではパスワードなしのユーザ`myuser`とする。

コンテナ内ユーザの特定ディレクトリにホストの公開鍵をマウントすれば、
公開鍵認証ができる。

### テスト
ここではテストのため、ホストの`0.0.0.0:2222`にSSHサーバのポートを割り当てている。

ホストから秘密鍵`YOUR_PRIVATE_KEY`を使ってこのSSHサーバにログインできるか確認する。

```sh
ssh localhost -p 2222 -l myuser -i ~/.ssh/YOUR_PRIVATE_KEY
```

起動、終了は`docker-compose up -d`、`docker-compose down`でよい。
確認後はポートの割り当ては不要なので削除する。

## HTTPプロキシを立てる
HTTPプロキシを立てる。[Squid](https://github.com/squid-cache/squid)を使う。Dockerイメージは[sameersbn/squid](https://github.com/sameersbn/docker-squid)を使う。

```
Docker Image Tag: 3.5.27-2
Commit ID: 924b0855440442a4be330ef4dba7a85681e9a49d
```

### docker-compose.yml

```yaml
version: '3.8'
services:
  sshd:
    image: maltyxx/sshd
    build: https://github.com/maltyxx/docker-sshd.git
    restart: always
    command: 'myuser::1000'
    volumes:
      - './YOUR_PUBLIC_KEY.pub:/home/myuser/.ssh/keys/YOUR_PUBLIC_KEY.pub:ro'

  http_proxy:
    image: sameersbn/squid
    restart: always
    ports:
      - '3128:3128'
    volumes:
      - './squid/squid.conf:/etc/squid/squid.conf:ro'
    depends_on:
      - 'sshd'
```

### squid/squid.conf

```squid
acl all src all

acl Safe_ports port 22
http_access deny !Safe_ports

acl sshd dstdomain sshd
http_access allow sshd
http_access deny all

http_port 3128

```

### テスト

```sh
ssh sshd -p 22 -l myuser \
  -i ~/.ssh/YOUR_PRIVATE_KEY \
  -o ProxyCommand='nc -X connect -x localhost:3128 %h %p'
```

ProxyCommand部の設定方法はOSによって変わるので注意（別記事を参照、[GPU PC Setupの最後におまけとして](https://blog.aoirint.com/entry/2020/gpu_pc_setup/#h_entry_2020_gpu_pc_setup_index_md_ssh%E3%81%AE%E3%83%97%E3%83%AD%E3%82%AD%E3%82%B7%E8%A8%AD%E5%AE%9A_18)）。

ここで指定している`localhost`はsshコマンドを実行しているクライアントのlocalhost。

`ssh`の直後の`sshd`の名前解決はSSHサーバ側で行われる（ProxyCommandの`%h`に入る）。


## リバースプロキシを立てる
リバースプロキシを立てる。nginxを使う。Dockerイメージは[_/nginx](https://hub.docker.com/_/nginx)を使う。


### docker-compose.yml

```yaml
version: '3.8'
services:
  sshd:
    image: maltyxx/sshd
    build: https://github.com/maltyxx/docker-sshd.git
    restart: always
    command: 'myuser::1000'
    volumes:
      - './YOUR_PUBLIC_KEY.pub:/home/myuser/.ssh/keys/YOUR_PUBLIC_KEY.pub:ro'

  http_proxy:
    image: sameersbn/squid
    restart: always
    volumes:
      - './squid/squid.conf:/etc/squid/squid.conf'
    depends_on:
      - 'sshd'

  nginx:
    image: nginx
    restart: always
    ports:
      # - '127.0.0.1:8000:80'
      - '8000:80'
    volumes:
      - './nginx/default.conf:/etc/nginx/templates/default.conf.template'
    depends_on:
      - 'http_proxy'
```

### nginx/default.conf

```nginx

upstream @http_proxy {
  server http_proxy:3128;
}

server_tokens off;

server {
  # listen 80;
  # server_name proxy.example.com;

	proxy_set_header X-Forwarded-Server $host;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header X-Forwarded-Proto $scheme;
	proxy_redirect off;

  proxy_pass http://@http_proxy;

}

```



### おまけ：Stream Proxy
`stream`という機能でTCP/UDPプロキシが張れる。
しかしTCP/UDPの仕様上リバースプロキシはできないと思われるので
今回は使わなかった。

- [Module ngx_stream_core_module | nginx.org](https://nginx.org/en/docs/stream/ngx_stream_core_module.html)
- [NGINX 1.9が汎用TCPサーバとして使えるようになっていた件 - Qiita](https://qiita.com/dseg/items/75bf517738a1d8b2d036#%E3%81%AF%E3%81%98%E3%82%81%E3%81%AB)

#### nginx/nginx.conf

```nginx
worker_processes auto;

error_log /var/log/nginx/error.log info;

events {
    worker_connections  1024;
}

stream {
  upstream @sshd {
    server sshd:22;
  }

  server {
    listen 80;
    proxy_pass @sshd;
  }

}
```

#### docker-compose.yml

```yaml
version: '3.8'
services:
  sshd:
    image: maltyxx/sshd
    build: https://github.com/maltyxx/docker-sshd.git
    restart: always
    command: 'myuser::1000'
    volumes:
      - './YOUR_PUBLIC_KEY.pub:/home/myuser/.ssh/keys/YOUR_PUBLIC_KEY.pub:ro'

  nginx:
    image: nginx
    restart: always
    ports:
      - '8000:80'
    volumes:
      - './nginx/nginx.conf:/etc/nginx/nginx.conf:ro'
    depends_on:
      - 'sshd'
```

#### テスト

```sh
ssh localhost -p 8000 -l myuser -i ./YOUR_PRIVATE_KEY
```


- [multiplexing - Can nginx serve SSH and HTTP(S) at the same time on the same port? - Super User](https://superuser.com/questions/1135208/can-nginx-serve-ssh-and-https-at-the-same-time-on-the-same-port)

- [Run Non-SSL Protocols on the Same Port as SSL in NGINX 1.15.2](https://www.nginx.com/blog/running-non-ssl-protocols-over-ssl-port-nginx-1-15-2/)
- [nginx でSSHをプロキシして転送する 443 の再利用も可能 - それマグで！](https://takuya-1st.hatenablog.jp/entry/2019/09/27/035208)
- [sslh でport443 を有効活用して、sshもhttpsも同時に待ち受けする。 - それマグで！](https://takuya-1st.hatenablog.jp/entry/2016/10/14/144244)
