---
canonical_url: ./
title: Pythonを追加するDockerfile
# og_image:
# twitter_card: summary_large_image
og_description: Pythonを追加するDockerfile
date: '2020-12-08 22:30:00'
draft: false
category: Docker
tags:
  - Docker
  - Python
---

# Pythonを追加するDockerfile

pyenvは部品を使うだけで最終的には削除します（Pythonは/usr/localに導入）

```dockerfile
FROM ubuntu:bionic


ARG DEBIAN_FRONTEND=noninteractive
ARG PYTHON_VERSION=3.9.0
ARG PYTHON_ROOT=/usr/local
ARG PYENV_ROOT=/tmp/.pyenv
ARG PYBUILD_ROOT=/tmp/python-build

RUN apt update && apt install -y \
    build-essential \
    libssl-dev \
    zlib1g-dev \
    libbz2-dev \
    libreadline-dev \
    libsqlite3-dev \
    wget \
    curl \
    llvm \
    libncurses5-dev \
    libncursesw5-dev \
    xz-utils \
    tk-dev \
    libffi-dev \
    liblzma-dev \
    python-openssl \
    git \
  && git clone https://github.com/pyenv/pyenv.git $PYENV_ROOT \
  && PREFIX=$PYBUILD_ROOT $PYENV_ROOT/plugins/python-build/install.sh \
  && $PYBUILD_ROOT/bin/python-build -v $PYTHON_VERSION $PYTHON_ROOT \
  && rm -rf $PYBUILD_ROOT $PYENV_ROOT

```
