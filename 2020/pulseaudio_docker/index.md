---
title: 'PulseAudioをDockerで使う'
# og_image:
# twitter_card: summary_large_image
og_description: 'ホストのPulseAudioにDockerからアクセスします'
date: '2020-11-14 14:30:00'
draft: true
category: 記事
tags:
  - PulseAudio
  - Docker
---
# PulseAudioをDockerで使う
Raspberry Piを使ったデバイスを作るときや、
デスクトップアプリケーションを動かしたいとき、
Dockerコンテナから音を鳴らしたい、ということが
よくあるのでメモしておきます。

ここでは、UbuntuまたはRaspberry Pi OS上でPulseAudio・Dockerを一般ユーザで動かすことを前提に進めていきます。


## いろいろ設定が必要
今のところ、PulseAudioをDocker内から使うには
ホスト側でそれなりの設定が必要になるほか、
Docker内ユーザのUID・GIDをホストユーザのUID・GIDと同じにすることが
必要であるように思っています。
ホスト側のセキュリティを緩めればできるかもですが、
調査が必要なので今回は扱いません。


## ホスト側PulseAudioの設定
以下の内容は、デスクトップ環境（Ubuntu Desktop）などではすでに設定されているかもしれません。
Raspberry Pi OS Lite（2020-08-20）では設定されていなかったようなので、手順を書いておきます。

まずはホストに`PulseAudio`をインストールしておきます。

```bash
sudo apt install pulseaudio
```

次はホストでのPulseAudioの起動設定です。
ホストOSが起動してから、手動でPulseAudioを起動することはあまり考えられないので、
OSの起動時にPulseAudioが動いていた方がいいでしょう。

OS起動時などのプロセス（サービス）の自動起動は`systemd`で管理されていて、
`systemctl`コマンドでサービスの起動停止や状態管理をします。
サービスの設定ファイルである`*.service`ファイルは`/etc/systemd/system`に配置されています。

しかしここは当然`root`ユーザしかいじれないので、一般ユーザが同様に自動起動をするためのコマンドとして、`systemctl --user`コマンドがあります。
このコマンドを使用するには、あらかじめ`root`権限で設定が必要なようです。

```bash
sudo loginctl enable-linger $USER
```

これでユーザ名`$USER`（現在のシェルのユーザ名）で`systemctl --user`が使えるようになるはずです。

`systemd`の一般ユーザ用の設定は`~/.config/systemd/user`に配置します。

- [Pulseaudio via systemd --user | Gist by @kafene](https://gist.github.com/kafene/32a07cac0373409e31f5bfe981eefb19)

こちらのGistの設定ファイルが有用だったので使わせてもらいます。

~/.config/systemd/user/pulseaudio.service
```systemd
[Unit]
Description=Pulseaudio Sound Service
Requires=pulseaudio.socket

[Service]
Type=notify
ExecStart=/usr/bin/pulseaudio --verbose --daemonize=no
ExecStartPost=/usr/bin/pactl load-module module-alsa-sink
Restart=on-failure

[Install]
Also=pulseaudio.socket
WantedBy=default.target
```

~/.config/systemd/user/pulseaudio.socket
```systemd
[Unit]
Description=Pulseaudio Sound System

[Socket]
Priority=6
Backlog=5
ListenStream=%t/pulse/native

[Install]
WantedBy=sockets.target
```

PulseAudioを手動で起動するときは`pulseaudio --start`を使いますが、
ここではkillした上でサービスの自動起動を有効にし、`systemd`を使ってPulseAudioを再起動します。

```
pulseaudio --kill

systemctl --user daemon-reload
systemctl --user enable pulseaudio.service
systemctl --user enable pulseaudio.socket

systemctl --user start pulseaudio.service
```


## Docker側の設定
まず、Dockerコンテナの作業ユーザを`root`ではなく一般ユーザにします。
また、ここではホストのPulseAudioと通信するため、
Dockerコンテナを作成したユーザと同じ`PID`・`GID`をもつユーザにします。

DockerイメージにはPulseAudioに加えて、
音声ファイル再生コマンド`play`を使うため、`sox`をインストールしています。

PulseAudioが起動しているとき、
ユーザごとの`${XDG_RUNTIME_DIR}/pulse/native`(`%t/pulse/native`、`/run/user/$UID/pulse/native`)に
PulseAudioのプロセスと通信するUnixソケットファイルが生成されます。
また、認証情報として`~/.config/pulse/cookie`が生成されます。
今回はDockerコンテナにこれらのファイルをマウントすることでPulseAudioと通信していきます。

### 起動コマンドを使った設定
Dockerfile
```dockerfile
RUN apt update && apt install -y \
  pulseaudio \
  sox \
	libsox-fmt-all
```

ビルドします。
```bash
docker build . -t aoirint/pulseaudio
```

```bash
docker run \
  -u "$(id -u):$(id -g)"
  -v
  aoirint/pulseaudio

```

### docker-composeを使った設定


### エントリポイントを使った設定
一応、エントリポイント（`ENTRYPOINT`、Dockerコンテナ起動時に毎回実行されるコマンド）を
使う方法を書いておきます。
ここでは、`gosu`を使って`CMD`（`ENTRYPOINT`の引数、またはデフォルトで実行されるコマンド）を
一般ユーザで実行するようにします。

Dockerfile
```dockerfile
RUN apt update && apt install -y \
  pulseaudio \
  sox \
	libsox-fmt-all \
  gosu

ADD ./docker-entrypoint.sh /
ENTRYPOINT [ "/docker-entrypoint.sh" ]
```

docker-entrypoint.sh（ホスト側で実行権限を付与）
```shell
#!/bin/sh

```
