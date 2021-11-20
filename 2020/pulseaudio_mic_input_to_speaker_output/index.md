---
title: 'PulseAudioでマイク入力をスピーカから出力する'
# og_image:
# twitter_card: summary_large_image
og_description: 'PulseAudioでマイク入力をスピーカから出力する'
date: '2020-10-04 03:40:00'
updated: '2021-08-16 15:00:00'
draft: false
category: PulseAudio
tags:
  - PulseAudio
  - 'Virtual Audio'
---
# PulseAudioでマイク入力をスピーカから出力する

まずはPCに接続されている音声入力デバイスのIDを確認する。
```sh
pactl list short sources
# pactl list sources
```

```
3	alsa_output.pci-0000_00_1f.3.analog-stereo.monitor	module-alsa-card.c	s16le 2ch 44100Hz	RUNNING
4	alsa_output.pci-0000_01_00.1.hdmi-stereo.monitor	module-alsa-card.c	s16le 2ch 44100Hz	RUNNING
5	alsa_output.usb-GeneralPlus_USB_Audio_Device-00.iec958-stereo.monitor	module-alsa-card.c	s16le 2ch 44100Hz	RUNNING
6	alsa_input.usb-GeneralPlus_USB_Audio_Device-00.analog-mono	module-alsa-card.c	s16le 1ch 44100Hz	RUNNING
```

ここでは、以下のようになっている。

|ID|種類|
|:--|:--|
|3|本体のヘッドホン端子|
|4|HDMI|
|5|USBサウンドカードのヘッドホン端子|
|6|USBサウンドカードのマイク端子|

以下のコマンドで、ID=6のUSBサウンドカードのマイク端子に入力されている音声をスピーカにループバックするモジュールがロードされる。

```sh
pacmd load-module module-loopback source=6
```

名前を使って指定することもできる。

```sh
pacmd load-module module-loopback source=alsa_input.usb-GeneralPlus_USB_Audio_Device-00.analog-mono
```

必要に応じて、`pavucontrol`を開いて出力先デバイスを設定する。

以下のコマンドで、ループバックモジュールをアンロードする。

```sh
pacmd unload-module module-loopback
```
