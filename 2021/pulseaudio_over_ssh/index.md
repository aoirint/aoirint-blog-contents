---
title: PulseAudioでSSHサーバ側の音声をクライアント側で再生する
date: '2021-10-09T20:00:00+09:00'
draft: false
channel: 技術ノート
category: PulseAudio
tags:
  - PulseAudio
---

# PulseAudioでSSHサーバ側の音声をクライアント側で再生する

RDPと違い、VNCでは接続先の音声を送れないことがある。

接続先のデスクトップ（SSH・VNCサーバ）で再生された音声をSSH経由で送信し、SSH・VNCクライアント側で再生できるようにする。

- [https://raspberrypi.stackexchange.com/questions/8621/how-to-set-up-a-pulseaudio-sink](https://raspberrypi.stackexchange.com/questions/8621/how-to-set-up-a-pulseaudio-sink)
- [https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/#module-tunnel-sink-new](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/#module-tunnel-sink-new)
  - `module-tunnel-sink-new`の実装はすでに`module-tunnel-sink`で動くようになっている（`pulseaudio 13.99.1`）

## 音声を受け取る側（SSH・VNCクライアント、PulseAudioサーバ）

### /etc/pulse/default.pa

```pulseaudio
load-module module-native-protocol-tcp auth-ip-acl=127.0.0.0/8

load-module module-null-sink sink_name=DummyOutputRemote0 sink_properties=device.description="DummyOutputRemote0"
load-module module-loopback source=DummyOutputRemote0.monitor source_dont_move=true
```

DummyOutputRemote0に対して音声が送られてくる。

DummyOutputRemote0.monitorで入力として音声を拾うこともできる。

### ~/.ssh/config

```pulseaudio
Host your-server
    # VNC
    LocalForward 15900 localhost:5900
    # PulseAudio
    RemoteForward 14713 localhost:4713
```

## 音声を送る側（SSH・VNCサーバ、PulseAudioクライアント・サーバ＝中継用）

### /etc/pulse/default.pa

```pulseaudio
load-module module-tunnel-sink sink_name=Remote server=tcp:127.0.0.1:14713 sink=DummyOutputRemote0
```

PulseAudio再起動（`pulseaudio -k`）後、
接続先のデスクトップ上で音声出力デバイス（Sink）をDummyOutputRemote0に設定する。
DummyOutputRemote0が表示されない場合、どこか設定が間違っていると思われる。

- 電源メニューから音声デバイスを変更できるようにするGNOME拡張
  - [https://extensions.gnome.org/extension/906/sound-output-device-chooser/](https://extensions.gnome.org/extension/906/sound-output-device-chooser/)
