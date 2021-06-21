---
canonical_url: ./
title: 透過プロキシ（Ubuntu, go-transproxy）
date: '2021-06-21 18:10:00'
draft: false
category: プロキシ
tags:
  - プロキシ
  - Ubuntu
---

# 透過プロキシ（Ubuntu, go-transproxy）

## /etc/sudoers

```sudoers
Defaults        env_keep+="HTTP_PROXY"
Defaults        env_keep+="HTTPS_PROXY"
Defaults        env_keep+="NO_PROXY"
```

## 環境変数

```bash
export HTTP_PROXY=http://proxy.example.com:8080/
export HTTPS_PROXY=http://proxy.example.com:8080/
export NO_PROXY=localhost,127.0.0.0/8,::1
```

NO_PROXYには必ずlocalhost/ローカルループバックアドレスを設定しておくこと（無限ループする）。

デスクトップの場合はGUI設定でOK。


## /usr/local/bin/transproxy

- [https://github.com/wadahiro/go-transproxy](https://github.com/wadahiro/go-transproxy)


## /usr/local/bin/transproxy-start
```bash
#!/bin/bash

set -eu

# iptables -t nat -I OUTPUT -p tcp --dport 53 -j REDIRECT --to-ports 3131
# iptables -t nat -I OUTPUT -p udp --dport 53 -j REDIRECT --to-ports 3131
iptables -t nat -I OUTPUT -p tcp --dport 80 -j REDIRECT --to-ports 3129
iptables -t nat -I OUTPUT -p tcp --dport 443 -j REDIRECT --to-ports 3130

transproxy -disable-iptables # -dns-over-https-enabled
```


## /usr/local/bin/transproxy-stop
```bash
#!/bin/bash

set -eu

# iptables -t nat -D OUTPUT -p tcp --dport 53 -j REDIRECT --to-ports 3131
# iptables -t nat -D OUTPUT -p udp --dport 53 -j REDIRECT --to-ports 3131
iptables -t nat -D OUTPUT -p tcp --dport 443 -j REDIRECT --to-ports 3130
iptables -t nat -D OUTPUT -p tcp --dport 80 -j REDIRECT --to-ports 3129
```

iptablesのルールを書き換えるため、実行時にはroot権限が必要。

sudoで実行すれば、/etc/sudoersの設定から現在のシェルの環境変数が引き継がれる（-Eオプションでもいいが）。

transproxyは内部で環境変数からプロキシ設定を読み出し、透過プロキシサーバを立てる。


## 自動起動設定 /etc/systemd/system/transproxy.service

常時プロキシに接続する端末ではOS起動時に実行されるようにしておくと便利。

```systemd
[Unit]
Description=transproxy

[Service]
Environment=HTTP_PROXY=http://proxy.example.com:8080/
Environment=HTTPS_PROXY=http://proxy.example.com:8080/
Environment=NO_PROXY=localhost,127.0.0.0/8,::1
ExecStart=/usr/local/bin/transproxy-start
ExecStop=/usr/local/bin/transproxy-stop
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target

```
