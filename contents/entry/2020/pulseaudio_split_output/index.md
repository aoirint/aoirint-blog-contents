---
title: PulseAudioで特定のアプリケーションからの音声出力だけを分離する
# og_image:
# twitter_card: summary_large_image
og_description: PulseAudioで特定のアプリケーションからの音声出力だけを分離する
date: '2020-11-09 06:00:00'
updated: '2021-08-22 04:20:00'
draft: false
category: PulseAudio
tags:
  - PulseAudio
  - 'Virtual Audio'
---

# PulseAudioで特定のアプリケーションからの音声出力だけを分離する

以下のコマンドで仮想出力デバイス`DummyOutput0`と、そのループバック`Loopback from Monitor of DummyOutput0`が追加される。

```bash
pacmd load-module module-null-sink sink_name=DummyOutput0 sink_properties=device.description=DummyOutput0
pacmd load-module module-loopback source=DummyOutput0.monitor
```

`pavucontrol`または`PulseAudio Volume Control`の`Playback`から
目的のアプリケーションの出力先を`DummyOutput0`にする。
また、`Loopback from Monitor of DummyOutput0`の出力先を
希望のスピーカーにすることで音声出力を分離しつつ同時に視聴できる。

仮想出力デバイスとループバックを削除するには、以下のコマンドを実行する。

```bash
pacmd unload-module module-loopback
pacmd unload-module module-null-sink
```


## ログイン時に自動作成する
### /etc/pulse/default.pa

```pulseaudio
# Custom
load-module module-null-sink sink_name=DummyOutputApp sink_properties=device.description="DummyOutputApp"
load-module module-loopback source=DummyOutputApp.monitor

load-module module-null-sink sink_name=DummyOutputVoiceChat sink_properties=device.description="DummyOutputVoiceChat"
load-module module-loopback source=DummyOutputVoiceChat.monitor

load-module module-null-sink sink_name=DummyOutputGeneral0 sink_properties=device.description="DummyOutputGeneral0"
load-module module-loopback source=DummyOutputGeneral0.monitor

load-module module-null-sink sink_name=DummyOutputGeneral1 sink_properties=device.description="DummyOutputGeneral1"
load-module module-loopback source=DummyOutputGeneral1.monitor
```

### PulseAudioの再起動

```shell
pulseaudio --kill
pulseaudio --start
```
