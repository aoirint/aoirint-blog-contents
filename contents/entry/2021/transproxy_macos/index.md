---
canonical_url: ./
title: 透過プロキシ（macOS, go-transproxy）
date: '2021-06-21 18:40:00'
draft: false
category: プロキシ
tags:
  - プロキシ
  - macOS
---

# 透過プロキシ（macOS）

## 環境変数

シェル起動時にネットワーク設定からプロキシ設定を環境変数に読み出すようにしている場合、GUIで設定すればOK。

```bash
export HTTP_PROXY=http://proxy.example.com:8080/
export HTTPS_PROXY=http://proxy.example.com:8080/
export NO_PROXY=localhost,127.0.0.0/8,::1
```

NO_PROXYには必ずlocalhost/ローカルループバックアドレスを設定しておくこと（無限ループする）。


## ビルド
- [https://github.com/wadahiro/go-transproxy](https://github.com/wadahiro/go-transproxy)

```shell
git clone https://github.com/wadahiro/go-transproxy.git
cd go-transproxy/
make

ls go-transproxy/bin/transproxy

# check network device name
ifconfig
```

## ~/transproxy/pf.conf

MacBookの無線LANカード：`en0`

Mac miniなどの場合はNIC名が異なる場合があるので注意。

```pf
Packets = "proto tcp from en0 to {!192.168.0.0/16}"
rdr pass log on lo0 $Packets port 80 -> 127.0.0.1 port 3129
rdr pass log on lo0 $Packets port 443 -> 127.0.0.1 port 3130
pass out on en0 route-to lo0 inet $Packets port {80, 443} keep state
```

## 有効化
```shell
sudo pfctl -ef ~/transproxy/pf.conf
transproxy -disable-iptables
```

## 無効化
```shell
# stop transproxy process (Ctrl+C)

# reset pf configs
sudo pfctl -df /etc/pf.conf
```

- [Macでプロキシとの戦いに疲れたので、透過型プロキシを導入してみた - Qiita](https://qiita.com/informationsea/items/094146d0a811f3edc96b)
