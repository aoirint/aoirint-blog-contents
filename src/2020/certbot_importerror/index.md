---
# moved from https://aoirint.hatenablog.com/entry/2020/03/27/100941
title: 'Let''s Encrypt certbot Error Ubuntu（ImportError: cannot import name ''constants''）'
date: '2020-03-27T10:09:41+09:00'
draft: false
channel: 技術ノート
category: Let's Encrypt
tags:
- Let's Encrypt
- Ubuntu
---
# Let's Encrypt certbot Error Ubuntu（ImportError: cannot import name 'constants'）

間違えてOSのパッケージリポジトリからcertbotを入れてしまった.

コマンド

```sh
sudo certbot --nginx
```

エラー1（初期状態）

```plain
ImportError: cannot import name 'constants'
```

エラー2（`pip3 uninstall certbot`の後）

```plain
AttributeError: module 'acme.challenges' has no attribute 'TLSSNI01Response'
```

- [Certbot - Ubuntubionic Nginx](https://certbot.eff.org/lets-encrypt/ubuntubionic-nginx)

```sh
sudo pip3 uninstall certbot acme
sudo apt purge certbot python-certbot-nginx

# HERE: register official ppa repository (see official guide)

sudo apt install certbot python-certbot-nginx
```
