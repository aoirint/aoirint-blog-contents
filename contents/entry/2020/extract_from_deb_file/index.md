---
canonical_url: ./
title: Debianパッケージから内容物を抽出する
# og_image:
# twitter_card: summary_large_image
og_description: Debianパッケージから内容物を抽出する
date: '2020-12-08 22:40:00'
draft: false
category: Ubuntu
tags:
  - Ubuntu
  - Debian
---

# Debianパッケージから内容物を抽出する

- [Debian -- buster の bash パッケージに関する詳細](https://packages.debian.org/buster/bash)
    - http://ftp.jp.debian.org/debian/pool/main/b/bash/bash_5.0-4_amd64.deb

```bash
wget http://ftp.jp.debian.org/debian/pool/main/b/bash/bash_5.0-4_amd64.deb
mkdir bash
tar xvf bash_5.0-4_amd64.deb -C bash/
cd bash
mkdir data
tar xvf data.tar.xz -C data/

cd data
# here is root directory
```
