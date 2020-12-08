---
canonical_url: ./
title: Dockerイメージから内容物を抽出する
# og_image:
# twitter_card: summary_large_image
og_description: Dockerイメージから内容物を抽出する
date: '2020-12-08 22:30:00'
draft: false
category: Docker
tags:
  - Docker
---

# Dockerイメージから内容物を抽出する

- [docker save | Docker Documentation](https://docs.docker.com/engine/reference/commandline/save/#save-an-image-to-a-targz-file-using-gzip)

```bash
docker pull alpine:3
docker save -o out.tar alpine:3
mkdir out
tar xvf out.tar -C out/
cd out/6709f754bd0ccbbea9a7481e92772a494cca1543b3421978edff62bc5de16662
tar xvf layer.tar -C layer/
cd layer
```
