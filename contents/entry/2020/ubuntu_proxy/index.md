---
canonical_url: ./
title: 'Ubuntu プロキシ設定'
# og_image:
# twitter_card: summary_large_image
og_description: 'Ubuntuにプロキシを設定する'
date: '2020-10-02 15:40:00'
updated: '2021-06-21 18:50:00'
draft: false
category: プロキシ
tags:
  - Ubuntu
  - Proxy
  - 'PC Setup'
---
# Ubuntu プロキシ設定

## User (Desktop)
Settings > Network > Network Proxy > Manualに設定する。
自動的に環境変数HTTP_PROXY, HTTPS_PROXYにスキームが追加された状態で設定される。

- HTTP Proxy: `proxy` `port`
- HTTPS Proxy: `proxy` `port`
- Ignore Hosts: `localhost, 127.0.0.0/8, ::1`（プライベートネットワーク・組織内ネットワークのIPアドレス範囲またはドメインも追加する）

curlやwget、pipなど主要コマンドは
注意点として、別ユーザとしてコマンドを実行すると環境変数が引き継がれない。
例えばsudo curlしたときにプロキシに接続しにいかない。

## sudoers environment keep
sudoでコマンドを実行したとき、実行時シェルに設定されている環境変数を引き継ぐようにする。
sudoersは書き込み禁止になっているためvisudoで編集する。

```sh
# run as root
visudo -f /etc/sudoers
```

```sudoers
Defaults env_keep+="HTTP_PROXY"
Defaults env_keep+="HTTPS_PROXY"
Defaults env_keep+="NO_PROXY"
Defaults env_keep+="EDITOR"
```

## /etc/environment
システム全体の環境変数として設定される。影響範囲が大きいので注意。
デスクトップユーザではSettings側の設定（Disabledなら設定されない）が優先されるようだった。

/etc/environment
```
HTTP_PROXY=http://proxy:port
HTTPS_PROXY=http://proxy:port
NO_PROXY=localhost, 127.0.0.0/8, ::1
```

## apt
/etc/apt/apt.conf
```
Acquire::http::proxy "http://proxy:port";                             
Acquire::https::proxy "http://proxy:port";
```

## Snap
/etc/systemd/system/snapd.service.d/override.confにsystemdの設定ファイルを作成する。

```sh
# run as root
systemctl edit snapd.service # edit service file here
systemctl daemon-reload
systemctl restart snapd.service

# run as user
snap install ...
```

/etc/systemd/system/snapd.service.d/override.conf
```systemd
[Service]
Environment=http_proxy=http://proxy:port
Environment=https_proxy=http://proxy:port
```

* [社内Proxyに阻まれSnapでパッケージ管理できないあなたへ : サイコロイドの備忘ログ](http://blog.livedoor.jp/tamanooboshi/archives/31598849.html "社内Proxyに阻まれSnapでパッケージ管理できないあなたへ : サイコロイドの備忘ログ")
* [Set snapd proxy via core configuration - snapd - snapcraft.io](https://forum.snapcraft.io/t/set-snapd-proxy-via-core-configuration/467/21 "Set snapd proxy via core configuration - snapd - snapcraft.io")


## 透過プロキシ

プロキシ非対応のHTTPアプリケーションでもプロキシを通過させるようにする透過プロキシを利用すると便利。

- [透過プロキシ（Ubuntu, go-transproxy）](https://blog.aoirint.com/entry/2021/transproxy_ubuntu/)
