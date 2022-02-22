---
# moved from https://aoirint.hatenablog.com/entry/2020/05/19/123527
title: Ubuntu上のデスクトップ音声出力をZoomの音声入力にする（PulseAudio）
date: '2020-05-19 12:35:27'
draft: false
channel: 技術ノート
category: Remote
tags:
- Remote
- VirtualDevice
---
# Ubuntu上のデスクトップ音声出力をZoomの音声入力にする（PulseAudio）

## Environment
- Ubuntu 18.04
- PulseAudio

## スピーカー出力を仮想マイク入力にする

  `pavucontrol`を実行し、`Input Devices`のタブを開く。`Show:`を`All Input Devices`に設定すると、`Monitor of YOUR_SPEAKER`という項目があり、このデバイスからの読み取りはスピーカーへの出力がループバックされたものになっている（`YOUR_SPEAKER`への出力＝`Monitor of YOUR_SPEAKER`からの入力。

しかしここで音声を入力するアプリケーションとして仮にZoomを開いても、Microphoneの選択肢に`Monitor of YOUR_SPEAKER`は現れない。回避策として、Microphoneを`Same as System`（デフォルトの音声入力デバイス）にし、Input Devicesで`Monitor of YOUR_SPEAKER`の`Set as Fallback`（緑のチェックマーク、デフォルトの音声入力デバイス）を有効にすることでZoomのマイクとしてスピーカー出力を渡すことができた。

このままでは他の人から送られてきた音声が自分のコンピュータの中でループ（Zoom Output→YOUR_SPEAKER→Monitor of YOUR_SPEAKER→Zoom Input）してしまうんじゃないかと思うが、少なくともZoom上ではそうはならなさそう（PC1のマイク→Zoom→PC2の音声出力→Monitor of PC2 Speaker→Zoom→PC1の音声出力 とはならなかった？）？　これはハウリング防止フィルタが効いてるのかどうなのか..

Zoomの音域系フィルタが効いているのか音楽は若干変になる（もともと声を送るためのチャンネルだし。きれいな音声を送るにはShare Screenを使う必要ありか）。

Zoomの入力デバイスをpavucontrolから直接変えられないのはZoom側でなにか固定してるのか、チャンネル数とかフォーマット対応してないみたいなことなのか..

前回のデスクトップ映像の出力と組み合わせた場合、映像と音声の同期がとれないと思われるので、お気持ちでなんとかするか、あるいはffmpegで同時に送り出すことまでを保証する、くらいはできるのだろうか..

- https://aoirint.hatenablog.com/entry/2020/05/15/185853

## 挿入音声の再生デバイス分離
Zoomの音声を出力するデバイスと挿入音声を出力するデバイスを分離して、目的の音声だけ送り出せるようにしてみる。つまり、Zoomの音声出力は`YOUR_SPEAKER`のままにして、挿入音声を仮想音声出力デバイスに出力するようにする。

```sh
pacmd load-module module-null-sink sink_name=DummyOutput0 sink_properties=device.description=DummyOutput0
# Or
pacmd load-module module-null-sink sink_name=DummyOutput0
pacmd update-sink-proplist DummyOutput0 device.description=DummyOutput0
# ---

pacmd unload-module module-null-sink
```

これでデフォルトの音声入力デバイスを`DummyOutput0`に設定すればZoom上に`DummyOutput0`への出力が送り出される。あとは挿入音声を流しているアプリケーションからの音声出力を`DummyOutput0`に送り出せばOK。

この状態で`DummyOutput0`に出力されている挿入音声を`YOUR_SPEAKER`で聞くには、`Monitor of DummyOutput0`をループバックして`YOUR_SPEAKER`に出力する。

```sh
pacmd load-module module-loopback source=DummyOutput0.monitor

pacmd unload-module module-loopback
```

## 物理マイクの入力をミックスする
ここでZoomの入力デバイスに送り出されるのは自分のコンピュータ上で出力された音声だけになっているので、物理マイクを接続していてもこれに入力された音声をZoomに送り出すことはできていない。物理マイクへの入力もZoom上に送り出せるようにする。

まず、物理マイクのPulseaudio上でのデバイス名を調べる。

```sh
pactl list short sources
pactl list sources
```

いずれかのコマンドで自分のマイクのデバイス名（`alsa_input.*`）、もしくはID（数値）がわかるはず。これをループバックする仮想デバイスを作成する。

```sh
pacmd load-module module-loopback source=YOUR_MIC_NAME_OR_ID
# Or
pacmd load-module module-loopback source=YOUR_MIC_NAME_OR_ID sink=DummyOutput0

pacmd unload-module module-loopback
```

音声の出力先（`sink`）は`pavucontrol`上で変更できる。


## Pulseaudioを再起動する

実験中に変な設定をしてしまったのかPulseaudioが応答しなくなってしまうことがあったので、再起動手順を書いておく。

まず、通常の手順と思われるもの。

```sh
pulseaudio --kill
pulseaudio --start
```

次に、強制的にプロセスキルして再起動するもの。

```sh
ps -e | grep pulseaudio
kill -9 PID

pulseaudio --start
# Or
pulseaudio -D # Daemon startup failed?
```

## 参考
- https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/
- https://unix.stackexchange.com/questions/174379/how-can-i-create-a-virtual-output-in-pulseaudio
- https://askubuntu.com/questions/403416/how-to-listen-live-sounds-from-input-from-external-sound-card
- https://unix.stackexchange.com/questions/77101/no-bluez-device-after-pactl-list-sources-short
- https://unix.stackexchange.com/questions/576785/redirecting-pulseaudio-sink-to-a-virtual-source

## ボツコマンド集
```sh
# 入力FIFOデバイスの作成、これに出力を与える方法がわからない
pacmd load-module module-pipe-source file=/tmp/DummyInput0.input source_name=DummyInput0 source_properties=device.description=DummyInput0

pacmd unload-module module-pipe-source
```

```sh
# 出力FIFOデバイスの作成、メモ
pacmd load-module module-pipe-sink file=/tmp/DummyOutput1.output sink_name=DummyOutput1 sink_properties=device.description=DummyOutput1

pacmd unload-module module-pipe-sink
```

```sh
# 出力先がRead-onlyなのでむり
ffmpeg -f pulse -i DummyOutput0.monitor -f pulse DummyInput
```

```sh
# 1回違う設定をしてからアンロードすると設定できる謎コマンド
pactl load-module module-echo-cancel sink_master=DummyOutput0
pactl unload-module module-echo-cancel
pactl load-module module-echo-cancel source_master=DummyOutput0.monitor

pactl unload-module module-echo-cancel
```
