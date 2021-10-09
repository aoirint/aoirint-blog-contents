---
title: PulseAudioでSSHサーバ側の音声をクライアント側で再生する
date: '2021-10-09 20:00:00'
draft: false
category: PulseAudio
tags:
  - PulseAudio
---

# PulseAudioでSSHサーバ側の音声をクライアント側で再生する

RDPと違い、VNCでは接続先の音声を送れないことがある。

接続先のデスクトップで再生された音声をSSH経由で送信し、ローカルで再生できるようにする。

- <https://raspberrypi.stackexchange.com/questions/8621/how-to-set-up-a-pulseaudio-sink>

## 音声を受け取る側（SSH・VNCクライアント）

### /pulse/default.pa

```pulseaudio
load-module module-native-protocol-tcp auth-ip-acl=127.0.0.0/16

load-module module-null-sink sink_name=DummyOutputRemote0 sink_properties=device.description="DummyOutputRemote0"
load-module module-loopback source=DummyOutputRemote0.monitor source_dont_move=true
```

DummyOutputRemote0に対して音声が送られてくる。

DummyOutputRemote0.monitorで入力として音声を拾うこともできる。


### ~/.ssh/config

```pulseaudio
Host your-host
    # VNC
    LocalForward 15900 localhost:5900
    # PulseAudio
    RemoteForward 14713 localhost:4713
```

## 音声を送る側（SSH・VNCサーバ）

### /pulse/default.pa

```pulseaudio
load-module module-tunnel-sink sink_name=Remote server=tcp:127.0.0.1:14713 sink=DummyOutputRemote0
```

PulseAudio再起動（`pulseaudio -k`）後、
接続先のデスクトップ上で音声出力デバイス（Sink）をDummyOutputRemote0に設定する。
DummyOutputRemote0が表示されない場合、どこか設定が間違っていると思われる。

- 電源メニューから音声デバイスを変更できるようにするGNOME拡張
    - <https://extensions.gnome.org/extension/906/sound-output-device-chooser/>
