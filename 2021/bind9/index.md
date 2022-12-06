---
title: bind9
date: '2021-06-09T01:30:00+09:00'
draft: false
channel: 技術ノート
category: Network
tags:
  - Network
  - DNS
  - bind9
---

# bind9

```shell
server$ sudo apt install bind9
```

## 使用中のDNSサーバの確認

クライアントに設定されているDNSサーバは`nmcli`コマンドで確認できる。

```shell
$ nmcli device show | grep DNS
IP4.DNS[1]:                             8.8.8.8
IP4.DNS[2]:                             8.8.4.4
IP4.DNS[3]:                             1.1.1.1
```

- [Command-line to list DNS servers used by my system - Ask Ubuntu](https://askubuntu.com/questions/152593/command-line-to-list-dns-servers-used-by-my-system)


## 用語

- [BIND DNSサーバー構築 | CentOSサーバー構築入門](http://cos.linux-dvr.biz/archives/category/bind-dns%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E6%A7%8B%E7%AF%89)

### コンテンツサーバ（権威サーバ）

あるゾーン（DNSにおける名前の管理単位）に関する名前解決要求（非再帰問い合わせ）を受け取り、自身が管理するゾーンならば名前解決結果を、委任情報を持つゾーンならば委任先の権威サーバ情報を応答する。

ルートゾーン（FQDN: `.`）を管理するルートサーバは、TLD（例えば `jp.`）の権威サーバへの委任情報を持つ。

ゾーン情報において、NSレコードにより指定される。

- [JPRS用語辞典｜ルートゾーン](https://jprs.jp/glossary/index.php?ID=0155)
- [JPRS用語辞典｜権威サーバー（権威DNSサーバー）](https://jprs.jp/glossary/index.php?ID=0145)
- [JPRS用語辞典｜ネームサーバー（DNSサーバー）](https://jprs.jp/glossary/index.php?ID=0157)

### キャッシュサーバ（フルサービスリゾルバ）

クライアントから名前解決要求（再帰問い合わせ）を受け取り、
コンテンツサーバへ反復的に名前解決要求（反復問い合わせ）を送ることで任意のドメイン名の名前解決を行う。

- [JPRS用語辞典｜フルサービスリゾルバー（キャッシュDNSサーバー）](https://jprs.jp/glossary/index.php?ID=0158)

例えば`hoge.example.jp.`というドメインの名前解決要求を受け取ったとき、
ルートサーバ`.`への非再帰問い合わせにより`jp.`権威サーバへの委任情報を取得する。
`jp.`権威サーバへの非再帰問い合わせにより`example.jp.`権威サーバへの委任情報を取得する。
`example.jp.`権威サーバへの非再帰問い合わせにより`hoge.example.jp.`のIPアドレスを取得する、のような流れで反復的に問い合わせを行うことで`hoge.example.jp.`の名前解決を行う。

ISPの提供するDNSサーバや会社などの組織内DNSサーバ、GoogleやCloudflareなどが提供するパブリックDNSがある。

- [ASCII.jp：DNSによる名前解決の仕組みを理解しよう (1/2)](https://ascii.jp/elem/000/000/463/463076/)


### 回送

プロキシのように、受け取った名前解決要求を他のDNSサーバに送り、
応答を返す。


## /etc/bind/named.conf.options

自身が管理していない情報（ゾーン、委任情報）に関する問い合わせについて、
他のDNSサーバ（フルサービスリゾルバ）に問い合わせを回送するように設定する。

- [「再帰問い合わせ」と「非再帰問い合わせ」の違いを教えてください：DNS Tips - ＠IT](https://www.atmarkit.co.jp/ait/articles/1510/28/news013.html)
- [BIND DNSサーバー構築 | CentOSサーバー構築入門](http://cos.linux-dvr.biz/archives/category/bind-dns%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E6%A7%8B%E7%AF%89)
- [フォワード（回送） : お父さんのためのDNS講座](http://ddns.blog.jp/archives/13568266.html)

### デフォルト設定ファイル

```bind
options {
        directory "/var/cache/bind";

        // If there is a firewall between you and nameservers you want
        // to talk to, you may need to fix the firewall to allow multiple
        // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

        // If your ISP provided one or more IP addresses for stable
        // nameservers, you probably want to use them as forwarders.
        // Uncomment the following block, and insert the addresses replacing
        // the all-0's placeholder.

        // forwarders {
        //      0.0.0.0;
        // };

        //========================================================================
        // If BIND logs error messages about the root key being expired,
        // you will need to update your keys.  See https://www.isc.org/bind-keys
        //========================================================================
        dnssec-validation auto;

        auth-nxdomain no;    # conform to RFC1035
        listen-on-v6 { any; };
};
```

### 変更点

```
forwarders {
        8.8.8.8;
        8.8.4.4;
        1.1.1.1;
};
```

- [初歩から解説！BINDでシンプルDNSサーバー構築【構築編】｜ 開発者ブログ ｜ 株式会社アイソルート](https://www.isoroot.jp/blog/2929/)


### 確認

```shell
server$ systemctl restart bind9.service
```

```shell
client$ dig @192.168.x.x aoirint.com
...
;; ANSWER SECTION:
aoirint.com.		3600	IN	A	185.199.108.153
aoirint.com.		3600	IN	A	185.199.111.153
aoirint.com.		3600	IN	A	185.199.109.153
aoirint.com.		3600	IN	A	185.199.110.153
...
```

- [digコマンドの使い方 - Qiita](https://qiita.com/hana_shin/items/e99f64a01f2632b7a719)


## /etc/bind/named.conf.local

DNSサーバの管理するゾーンの名前と対応する設定ファイル（ゾーンファイル）を設定する。

```bind
//
// Do any local configuration here
//

zone "example.com" IN {
        type master;
        file "example.com.zone";
};

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";
```

## /var/cache/bind/example.com.zone

ゾーン情報を設定する。

```zone
$ORIGIN example.com.
$TTL    3600
  @       IN     SOA   dns.example.com.   root.example.com. (
    2021060900      ; Serial
    3600            ; Refresh
    900             ; Retry
    604800          ; Expire
    86400           ; Minimum
)

        IN     NS    dns.example.com.
dns     IN     A     192.168.xx.xx

@       IN     A     192.168.xx.xx
local   IN     A     192.168.xx.xx
dev     IN     A     192.168.xx.xx
```

```shell
server$ systemctl restart bind9.service
```

```shell
client$ dig @192.168.x.x example.com
...
;; ANSWER SECTION:
example.com.		3600	IN	A	192.168.xx.xx
...


client$ dig @192.168.x.x local.example.com
...
;; ANSWER SECTION:
local.example.com.		3600	IN	A	192.168.xx.xx
...


client$ dig @192.168.x.x dev.example.com
...
;; ANSWER SECTION:
dev.example.com.		3600	IN	A	192.168.xx.xx
...
```

- [初歩から解説！BINDでシンプルDNSサーバー構築【構築編】｜ 開発者ブログ ｜ 株式会社アイソルート](https://www.isoroot.jp/blog/2929/)
- [サブドメインの運用と委任：実用 BIND 9で作るDNSサーバ（6）（2/3 ページ） - ＠IT](https://www.atmarkit.co.jp/ait/articles/0306/03/news002_2.html)
- [bind9で内部向けDNS構築　（Ubuntu14.04 Server） - Qiita](https://qiita.com/m1220/items/e2093f8f5dc1a6ce9802)
