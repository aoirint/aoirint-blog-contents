---
# moved from https://aoirint.hatenablog.com/entry/2020/09/06/213051
title: VBA-M on Docker
date: '2020-09-06 21:30:51'
draft: false
channel: 技術ノート
category: Emulator
tags:
- Emulator
- Docker
---
# VBA-M on Docker

[VisualBoyAdvance - M](https://github.com/visualboyadvance-m/visualboyadvance-m)をDocker上で動かすDockerfile（とrunコマンドのオプションセット）を作った。
要X Window System、Pulseaudio。Ubuntu Desktop 18.04（with NVIDIA Driver）で動作確認。

- [https://github.com/aoirint/vbam-docker](https://github.com/aoirint/vbam-docker)

## Dockerfile

```dockerfile
# for general machine
FROM ubuntu:bionic

# for nvidia-driver machine
# FROM nvidia/opengl:base-ubuntu18.04

ENV VERSION 2.1.4
ENV SHA1HASH bf6e452b53f47e2fbc4e6e41c92f567aa285cdbe

WORKDIR /vbam

RUN apt update \
  && apt -qq -y --no-install-recommends install \
  ca-certificates \
  tar \
  wget \
# -- from builddeps script
  build-essential \
  g++ \
  nasm \
  cmake \
  ccache \
  gettext \
  zlib1g-dev \
  libgl1-mesa-dev \
  libavcodec-dev \
  libavformat-dev \
  libswscale-dev \
  libavutil-dev \
  libswresample-dev \
  libgettextpo-dev \
  libpng-dev \
  libsdl2-dev \
  libsdl2-2.0 \
  libglu1-mesa-dev \
  libglu1-mesa \
  libgles2-mesa-dev \
  libsfml-dev \
  libsfml-graphics2.4 \
  libsfml-network2.4 \
  libsfml-window2.4 \
  libglew2.0 \
  libopenal-dev \
  libwxgtk3.0-dev \
  libwxgtk3.0 \
  libgtk2.0-dev \
  libgtk-3-dev \
  zip \
# sound driver to play sound on host
  pulseaudio \
# build
  && mkdir /vbam-build && cd /vbam-build \
  && wget -O vbam.tar.gz https://github.com/visualboyadvance-m/visualboyadvance-m/archive/v${VERSION}.tar.gz \
  && echo "${SHA1HASH} vbam.tar.gz" | sha1sum -c - \
  && mkdir src \
  && tar xf vbam.tar.gz -C src --strip-components 1 \
  && mkdir build && cd build \
  && cmake ../src \
  && make \
# copy to /usr/local/bin/
  && mv visualboyadvance-m /usr/local/bin/ \
# remove build environment
  && rm -r /vbam-build/
```

ベースイメージは（とりあえず）基本は`ubuntu:bionic`で、NVIDIAのGPUで動いてるマシンでビルドするときは`nvidia/opengl`にする。これをやらないと描画時に`libGL error: No matching fbConfigs or visuals found`を吐く（逆に`ubuntu:bionic`での動作確認はしていないが.. 設定で描画をOpenGL以外にすればどちらでも動きそう）。

頭の環境変数を適切なものに変えてバージョンを切り替える。

`ca-certificates`はwgetでGitHubからリリースを持ってくるときの証明書周りのエラー対策。`tar`、`wget`は持ってきて解凍する用。
`build-essential`から`zip`までは同梱されてる`./builddeps`（v2.1.4）を実行したときに呼び出されたコマンドから持ってきていて、
`pulseaudio`は音声再生用。

`#build`から先がビルド用のコマンドで、`make`の実行が終わった段階で`visualboyadvance-m`（実行ファイル）が生成される。
これをPATHの通った`/usr/local/bin`に移動して、（たぶん）いらないビルド環境を削除する。
ほんとはcmake、make、g++とかだけ入ったビルド用のイメージ上でビルドしたあと、実行ファイルだけ実行用のイメージにコピーしたほうがいいかもしれない。
依存関係が多くて（長くなって）面倒そうだったのでやってない。

## docker run

```sh
xhost + local:root
```

ホスト上でローカルのrootユーザに対してX Serverのアクセス制限を解除。
これでDockerコンテナに共有する`/tmp/.X11-unix`を介してDockerコンテナ上のrootユーザがX Clientを実行できるようになる。
rootだったらいい気がするが、一応全部のアクセス制限を復活させるときは`xhost -`。


```sh
sudo docker run -it --rm --name vbam \
  -e DISPLAY \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  --gpus all \
  --group-add $(getent group audio | cut -d: -f3) \
  -e PULSE_SERVER=unix:${XDG_RUNTIME_DIR}/pulse/native \
  -v ${XDG_RUNTIME_DIR}/pulse/native/:${XDG_RUNTIME_DIR}/pulse/native \
  -v ${HOME}/.config/pulse/cookie:/root/.config/pulse/cookie \
  -v ${PWD}/vbam:/vbam \
  -v ${PWD}/vbam-conf:/root/.config/visualboyadvance-m \
  vbam \
  visualboyadvance-m
```

`DISPLAY`と`/tmp/.X11-unix`はX Window用、
`--gpus all`は一応nvidia/openglをベースイメージにしたときのGPU指定を入れている。

`group-add`から`${HOME}.config/pulse/cookie`をマウントしてるところまでが音声再生用。ここは [OpenSiv3D を Docker 上で動かす - nekketsu^ω](https://nekketsuuu.github.io/entries/2017/12/04/opensiv3d-on-docker.html) を参考にした。ホスト上で鳴らしたサウンドと同じように`pavucontrol`から見える。

ホストの`vbam`ディレクトリをマウントしてVBA-Mからホスト側のファイルを参照できるようにする。
このディレクトリから読み込んで実行すれば`.sav`ファイルは同じディレクトリに保存されるのでホスト側に永続化される
（vbamディレクトリ以下にセーブファイルディレクトリを設定すれば同じく永続化される）。

それから、`vbam-conf`ディレクトリをマウントしてコンフィグをホスト側に永続化するようにする。
