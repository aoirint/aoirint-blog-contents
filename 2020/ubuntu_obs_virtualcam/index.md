---
# moved from https://aoirint.hatenablog.com/entry/2020/05/15/185853
title: Ubuntu上のOBSでVirtualCamを使う（デスクトップ映像を仮想カメラに送る）
date: '2020-05-15T18:58:53+09:00'
draft: false
channel: 技術ノート
category: Remote
tags:
- Remote
- VirtualDevice
---
# Ubuntu上のOBSでVirtualCamを使う（デスクトップ映像を仮想カメラに送る）

Zoomなどでデスクトップ画面をカメラ映像として共有するのに使える。

- 参考： [https://github.com/CatxFish/obs-virtual-cam/issues/17](https://github.com/CatxFish/obs-virtual-cam/issues/17)

## Environment

- Ubuntu 18.04
- FFmpeg 3.4.6-0ubuntu0.18.04.1
- v4l2loopback `#ed2b709`
- OBS Studio 25.0.8 `#14b0565`

obs-v4l2sink `#1ec3c8a` はOBS Studioがクラッシュして使えなかった。

## How to

- [https://github.com/obsproject/obs-studio](https://github.com/obsproject/obs-studio)
- [https://github.com/umlaeute/v4l2loopback](https://github.com/umlaeute/v4l2loopback)

```sh
# sudo apt install v4l2loopback-dkms # これはバージョンの問題で使えないかも

git clone https://github.com/umlaeute/v4l2loopback.git
cd v4l2loopback
make
sudo make install
sudo depmod -a
```

```sh
# 仮想カメラを作成（モジュールをロード）
sudo modprobe v4l2loopback devices=1 video_nr=10 card_label="OBS Cam"

# 映像転送サーバを立てる（OBS→仮想カメラ）
ffmpeg -an -listen 1 -i rtmp://127.0.0.1:1935/live -f v4l2 /dev/video10

# 仮想カメラを削除
sudo modprobe -r v4l2loopback
```

カメラデバイスの一覧を確認するには`sudo apt install v4l-utils`して以下のコマンド。

```sh
v4l2-ctl --list-devices
```

OBS Studioを開いてStreamingの設定をする。SettingsからStreamを選択。

- Service: Custom
- Server: rtmp://127.0.0.1:1935/live
- Stream Key:

一応、解像度を落とす設定をしたけどいらないかもしれない。Outputを開く。

- Output Mode: Advanced
- Rescale Output: 1280x720

Encoderはなにかかけてもffmpegが自動で認識して（rawvideoに）変換してくれそうだけど、ローカルでエンコードする必要ないはずなのではじめからrawvideoでStreamingしたいけど、選択肢がなさそう..。しかたないのでh264で転送した。

```
# ffmpegのログ

Stream mapping:
  Stream #0:1 -> #0:0 (h264 (native) -> rawvideo (native))
```

rawvideoの指定できるRecordingでも同じように設定できた（Recording to URLのURLにrtmpサーバを指定）けれど、Start RecordingするとOBS Studioがクラッシュしてしまった。

あとはOBS上でStart Streamingすれば仮想カメラへの配信が始まる。Stop Streamingするとffmpegが終了してしまうのだけれど、終了しないように設定できるのかな？

## 付録

### 一部のアプリケーション用の設定

ChromeやWebRTCで仮想カメラデバイスを使おうとすると問題が起こる（認識しない？）ことがあるみたい。Cheeseでも同じ。
これを回避するには仮想カメラの設定として`exclusive_caps`を指定する必要がある。
OutputされていないときにOutputフラグを消す、な感じみたいだけど、他のアプリケーションが映像を読み出してると思って使えないデバイス扱いしちゃうのかな？
Zoomの場合は逆にexclusive_capsを指定するとStart Videoできなくなった。

```sh
sudo modprobe v4l2loopback devices=1 video_nr=10 card_label="OBS Cam" exclusive_caps=1
```

### OBSを使わない方法

デスクトップ画面全体をffmpegで仮想カメラデバイスに送る場合のメモ。以下のコマンドが使える（ [https://github.com/CatxFish/obs-v4l2sink/issues/5](https://github.com/CatxFish/obs-v4l2sink/issues/5) ）。デスクトップ番号が0でない場合は`-i :0`を書き換える。

```sh
ffmpeg -f x11grab -r 15 -s 1920x1080 -i :0 -vcodec rawvideo -pix_fmt yuv420p -threads 0 -f v4l2 /dev/video0
```

### ボツ：obs-v4l2sinkを導入

最終的には動作しなかったが、メモ。

- [https://github.com/CatxFish/obs-v4l2sink](https://github.com/CatxFish/obs-v4l2sink)

```sh
sudo apt install qtbase5-dev
git clone --recursive https://github.com/obsproject/obs-studio.git
git clone https://github.com/CatxFish/obs-v4l2sink.git
cd obs-v4l2sink
mkdir build
cd build
cmake -DLIBOBS_INCLUDE_DIR="../../obs-studio/libobs" -DCMAKE_INSTALL_PREFIX=/usr ..
make -j4
sudo make install
```

obs-studioとobs-v4l2sinkのソースコードをクローンしてビルド・インストールしている。

インストール後にOBSを開くとメニューバーのToolsに `V4L2 Video Output` の項目が追加される。
