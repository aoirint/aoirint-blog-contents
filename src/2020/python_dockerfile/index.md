---
# moved from https://aoirint.hatenablog.com/entry/2020/03/16/143009
title: Python Dockerfile
date: '2020-03-16T14:30:09+09:00'
draft: false
channel: 技術ノート
category: Docker
tags:
- Docker
- Python
---
# Python Dockerfile

```docker
FROM python:3

RUN mkdir /code
WORKDIR /code
COPY requirements.txt /code/
RUN pip install -r requirements.txt
```
