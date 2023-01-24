---
title: '自宅ネットワークのbind9をdocker composeに移行した'
date: '2023-01-25T03:40:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Network
tags:
  - bind9
  - Network
  - 自宅サーバ
---
# 自宅ネットワークのbind9をdocker composeに移行した

- 自宅ネットワークのbind9をdocker compose化したので記録。
- これまでホストで直接動かしていたけれど、OOM Killerか何かに3,4日に1度くらいでキルされていたので不便だった
- これでdocker composeのrestart機構で再起動がかかるようになる
  - systemdのrestart機構ワカラナイ
- Dockerイメージ: [https://hub.docker.com/r/ubuntu/bind9](https://hub.docker.com/r/ubuntu/bind9)

## コード

### docker-compose.yml

```yaml
services:
  bind9:
    image: ubuntu/bind9:9.18-22.04_beta
    restart: always
    ports:
      - "0.0.0.0:53:53/udp"
    volumes:
      - "./volumes/bind_etc/named.conf.local:/etc/bind/named.conf.local:ro"
      - "./volumes/bind_etc/named.conf.options:/etc/bind/named.conf.options:ro"
      - "./volumes/bind_var_cache:/var/cache/bind:rw"
      - "./volumes/bind_var_lib:/var/lib/bind:rw"
    environment:
      - "TZ=Asia/Tokyo"
      - "BIND9_USER=bind" # UID=101
```

### volumes/bind_etc/named.conf.options

```named
acl my-network {
  10.0.0.0/8;
  172.16.0.0/12;
  192.168.0.0/16;
  localhost;
};

options {
  directory "/var/cache/bind";
  response-policy { zone "rpz"; };

  allow-query { any; };
  allow-recursion { my-network; };
  allow-query-cache { my-network; };

  dnssec-validation auto;

  forward only;
  forwarders {
    # ISP DNS1;
    # ISP DNS2;
    8.8.8.8;
    8.8.4.4;
    1.1.1.1;
  };
}
```

### volumes/bind_etc/named.conf.local

```named
zone "example.com" IN {
  type master;
  file "example.com.zone";
};

zone "rpz" {
  type master;
  file "rpz.zone";
  allow-query { none; };
};
```

### volumes/bind_var_cache/example.com.zone

```zone
$ORIGIN example.com.
$TTL    3600
@      IN     SOA   ns.example.com.   root.example.com (
    2023012500      ; Serial
    3600            ; Refresh
    900             ; Retry
    604800          ; Expire
    86400           ; Minimum
)

      IN     NS    ns.example.com.
ns     IN     A     127.0.0.1

@      IN     A     127.0.0.1
sub    IN     CNAME example.com.
```

### volumes/bind_var_cache/rpz.zone

```zone
$TTL    3600
@       IN     SOA   localhost. root.localhost. (
    2023012500      ; Serial
    3600            ; Refresh
    900             ; Retry
    604800          ; Expire
    86400           ; Minimum
)

        IN     NS    localhost.

; block malicious domains
malicious.example.com A 0.0.0.0
```

## 既知の問題

- libvirt dnsmasqが53/udpを先に取ると動かなくなりそう？

## ホストのbind9をアンインストール

```bash
sudo apt purge bind9
sudo rm -rf /var/cache/bind/
```
