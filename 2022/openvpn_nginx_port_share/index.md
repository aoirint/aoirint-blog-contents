---
title: OpenVPNとnginxをOpenVPNのPort Share機能で共存させる
date: '2022-10-19 00:50:00+09:00'
draft: false
channel: 技術ノート
category: Network
tags:
  - Network
  - OpenVPN
  - nginx
---
# OpenVPNとnginxをOpenVPNのPort Share機能で共存させる

TCP 443番ポートは、一般にHTTPSの通信に使われ、ファイアウォールやプロキシによる制限の強いネットワークでも通過できることが多い。
一方、WebサーバをホストしたいIPアドレスと、443番ポートでVPNをホストしたいIPアドレスが同一の場合、443番ポートの取り合い（ポート番号の重複）が起きてしまう。

OpenVPNには、OpenVPNが待ち受けるポートへの通信が、
VPNのパケットか、VPN以外のパケット（例えばnginxへのHTTPS通信）かを自動で判別して、
VPN以外のパケットを中継してくれるPort Share機能がある。

つまり、OpenVPNが443番（`0.0.0.0:443`）で待ち受け、
nginxは他の適当なポート（例えば`127.0.0.1:4443`）で待ち受けるようにして、
Port Share機能の転送先をnginxに設定すれば、443番で待ち受けるOpenVPNと、本来443番で待ち受けたいnginxとが共存できる。

UbuntuサーバへのOpenVPNの導入は、以下のDigitalOceanの記事が有用なので、参照されたい。

- OpenVPNで使用するサーバー証明書・クライアント証明書を発行するCAサーバをセットアップする記事: <https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-a-certificate-authority-ca-on-ubuntu-20-04>
- OpenVPNをセットアップする記事: <https://www.digitalocean.com/community/tutorials/how-to-set-up-and-configure-an-openvpn-server-on-ubuntu-20-04-ja>

OpenVPNのインストールにあたっての重要な点として、
`ステップ7 — OpenVPNの設定`の`（オプション）DNSの変更をプッシュして、VPNを介してすべてのトラフィックをリダイレクトする`を実行した上で、
`ステップ9 — ファイアウォールの設定`を手順通り実行することがある。
NAT下にあるなどの理由でufwの設定をしていなかったサーバーでも、新たにufwの設定をすることで、トラブルなくOpenVPNのインストールができる。
このステップを行わないと、VPN外へのパケットの転送が動作せず、VPN接続中のクライアントがインターネットに接続できない状況に陥る。

また、`（オプション）ポートとプロトコルの調整`で、待ち受けポートを443番、プロトコルをTCPにしておく。
OpenVPNの起動の際には、nginxとポート番号を取り合って起動に失敗しないように、nginxをあらかじめ停止し、ポート番号を書き換えておく。

## nginx 設定ファイル

`/etc/nginx/sites-enabled/`などにあるnginxの設定ファイルを書き換え、
4443番で待ち受けるようにする。

```nginx
server {
  listen 4443 http2 ssl;
}
```

大量にサイトをホストしている場合、コマンドやVSCodeなどでまとめて書き換えるとよい。

## /etc/openvpn/server/server.conf

OpenVPNのサーバー設定ファイルに`port-share`を追記する。

```openvpn
port 443
port-share 127.0.0.1 4443

proto tcp
```

## OpenVPNからnginxへのパケット転送の許可（ufw）

```shell
ufw allow from 10.8.0.0/24 to any port 4443
```

## nginx, OpenVPNの再起動

nginxおよびOpenVPNを再起動し、443番ポート宛てのHTTPS通信（nginx宛て）、443番ポート宛てのVPN接続（OpenVPN宛て）が正常に動作することを確認する。
