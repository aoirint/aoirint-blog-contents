---
title: PulseAudioで特定のアプリケーションからの音声出力だけを分離する
# og_image:
# twitter_card: summary_large_image
og_description: PulseAudioで特定のアプリケーションからの音声出力だけを分離する
date: '2020-11-09T06:00:00+09:00'
updated: '2021-08-22T21:40:00+09:00'
draft: false
channel: 技術ノート
category: PulseAudio
tags:
  - PulseAudio
  - 'Virtual Audio'
---

# PulseAudioで特定のアプリケーションからの音声出力だけを分離する

- [https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/#module-loopback](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/#module-loopback)

以下のコマンドで仮想出力デバイス`DummyOutput0`と、そのループバック`Loopback from Monitor of DummyOutput0`が追加される。

```bash
pacmd load-module module-null-sink sink_name=DummyOutput0 sink_properties=device.description=DummyOutput0
pacmd load-module module-loopback source=DummyOutput0.monitor source_dont_move=true
```

`pavucontrol`または`PulseAudio Volume Control`の`Playback`から
目的のアプリケーションの出力先を`DummyOutput0`にする。
また、`Loopback from Monitor of DummyOutput0`の出力先を
希望のスピーカーにすることで音声出力を分離しつつ同時に視聴できる。

`source_dont_move=true`は、デフォルトの入力デバイス変更時にLoopbackの入力デバイスも変更されてしまうのを防ぐオプション。
`pavucontrol`からも変更できなくなる。

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
load-module module-loopback source=DummyOutputApp.monitor source_dont_move=true

load-module module-null-sink sink_name=DummyOutputVoiceChat sink_properties=device.description="DummyOutputVoiceChat"
load-module module-loopback source=DummyOutputVoiceChat.monitor source_dont_move=true

load-module module-null-sink sink_name=DummyOutputGeneral0 sink_properties=device.description="DummyOutputGeneral0"
load-module module-loopback source=DummyOutputGeneral0.monitor source_dont_move=true

load-module module-null-sink sink_name=DummyOutputGeneral1 sink_properties=device.description="DummyOutputGeneral1"
load-module module-loopback source=DummyOutputGeneral1.monitor source_dont_move=true
```

### PulseAudioの再起動

```shell
pulseaudio --kill
pulseaudio --start
```

## デバイス接続時の入出力デバイス自動切り替えを無効化する

- [https://askubuntu.com/questions/1061414/how-to-disable-pulseaudio-automatic-device-switch](https://askubuntu.com/questions/1061414/how-to-disable-pulseaudio-automatic-device-switch)

新しい出力デバイスが検出されたとき、デフォルト出力デバイスが切り替わるのを防ぐ。

### /etc/pulse/default.pa

`load-module module-switch-on-port-available`、`load-module module-switch-on-connect`をコメントアウトする。

```pulseaudio
### Should be after module-*-restore but before module-*-detect
#load-module module-switch-on-port-available

### Use hot-plugged devices like Bluetooth or USB automatically (LP: #1702794)
#.ifexists module-switch-on-connect.so
#load-module module-switch-on-connect
#.endif
```

### PulseAudioの再起動

```shell
pulseaudio --kill
pulseaudio --start
```

## 電源メニューから音声デバイスを切り替えられるようにするGnome Shell Extension

- [https://extensions.gnome.org/extension/906/sound-output-device-chooser/](https://extensions.gnome.org/extension/906/sound-output-device-chooser/)
- [https://github.com/kgshank/gse-sound-output-device-chooser](https://github.com/kgshank/gse-sound-output-device-chooser)

PulseAudioを再起動するとデバイス一覧に古い項目が残り、選択するとGnome Desktopがフリーズするバグがある。
PulseAudio再起動後はAlt+F2でrコマンドを実行することで拡張機能がリロードされ、この問題を回避できる。
