---
title: 自宅サーバのWebサービスをVPSで中継するようにした
date: '2022-04-07 11:00:00'
draft: false
channel: レポート
category: Network
tags:
  - Network
  - 自宅サーバ
  - VPS
---
# 自宅サーバのWebサービスをVPSで中継するようにした

これまで自宅サーバへの外部からの接続には、
数年前から自宅のグローバルIPアドレスにフリーのドメインを紐づけて使っていたのですが、
3月末に更新を忘れてこのドメインを失効させてしまいました。

自宅サーバでは、主に個人用のWebサービスや身内用のゲームサーバが動いています。

同じドメインを再取得するには追加の料金がかかりますし、（若干用途が規約的に怪しいこともあり）新しくドメインを取り直す気にもなれなかったので、
以前から契約していたVPSをTCP/UDPプロキシとして使ってみることにしました。

この記事では、nginx.confのhttpディレクティブとstreamディレクティブそれぞれについて、拡張子でincludeを切り分けるようにする構成にしています。
基本的には、nginx.confに以下のように記述するイメージです。

```nginx
http {
        include /etc/nginx/conf.d/*.http;
}

stream {
        include /etc/nginx/conf.d/*.stream;
}
```

- 関連ツイート
    - <https://twitter.com/aoirint/status/1510309731642257408>
    - <https://twitter.com/aoirint/status/1511896279731040261>

また、この記事の内容には関係ありませんが、envsubstでconfigファイルに環境変数を注入する、nginxのDocker Hub公式Dockerイメージのtemplate機能を使用しているため、
nginx設定ファイルの拡張子がtemplateになっています。

上のツイートに関連して、`certbot --nginx`で自動生成される`/etc/letsencrypt/options-ssl-nginx.conf`を手動で`nginx.conf`に統合しているので、
以下の設定ではコメントアウトされているincludeがあります。

## 構成

- 自宅サーバ側：systemd + autosshによる自動SSH接続＆リモートポート転送
- VPS側：nginxによるHTTPリバースプロキシ・UDPプロキシ

## 自宅サーバ側

### ~/.ssh/config

```ssh
Host vps*
    HostName VPS_IPADDR
    Port VPS_SSH_PORT
    ServerAliveInterval 10
    ServerAliveCountMax 5
    TCPKeepAlive yes
    IdentitiesOnly yes
    IdentityFile ~/.ssh/vps

Host vps-forwarding-homeserver
    # SERVICE1
    RemoteForward 127.0.0.1:VPS_SERVICE1_PORT 127.0.0.1:LOCAL_SERVICE1_PORT

    # SERVICE2
    RemoteForward 127.0.0.1:VPS_SERVICE2_PORT 127.0.0.1:LOCAL_SERVICE2_PORT
```

### /etc/systemd/system/autossh-vps.service

```systemd
[Unit]
Description=AutoSSH VPS

[Service]
Type=simple
Restart=always
User=user
Group=user
WorkingDirectory=/home/user
ExecStart=/usr/bin/autossh -N vps-forwarding-homeserver

[Install]
WantedBy=multi-user.target
```

## VPS側

### HTTPリバースプロキシ（SSHポート転送使用）: myservice.http.template

```nginx
server {
    server_name myservice.example.com;

    client_max_body_size 500M;

    add_header Strict-Transport-Security 'max-age=15552000; includeSubDomains; preload';

    location / {
        #auth_basic "Auth required";
        #auth_basic_user_file /secrets/myservice.htpasswd;

        proxy_pass http://127.0.0.1:VPS_MYSERVICE_PORT;
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Host $host;
        proxy_set_header X-Forwarded-Server $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        send_timeout 3600s;
        proxy_connect_timeout 3600s;
        proxy_send_timeout 3600s;
        proxy_read_timeout 3600s;
        #proxy_max_temp_file_size 2048m;
    }

    location /.well-known/acme-challenge/ {
        root /letsencrypt-webroot;
    }

    listen 443 ssl;
    ssl_certificate /etc/letsencrypt/live/myservice.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myservice.example.com/privkey.pem;
    # include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
}

server {
    if ($host = myservice.example.com) {
        return 301 https://$host$request_uri;
    }

    server_name myservice.example.com;
    listen 80;
    return 404;
}
```

### UDPポート転送（SSHポート転送不使用）: myserviceudp.stream.template

```nginx
server {
        listen VPS_SERVICEUDP_PORT udp reuseport;

        proxy_pass HOME_IPADDR:HOME_SERVICEUDP_PORT;
        proxy_connect_timeout 3600s;
        proxy_timeout 3600s;
        #proxy_max_temp_file_size 2048m;
}
```
