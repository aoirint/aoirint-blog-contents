---
title: 'Ubuntu, CUDAを入れたあとにSteamが起動しない'
date: '2021-11-18 23:50:00'
draft: false
category: Game
tags:
  - Ubuntu
  - 'NVIDIA GPU'
---
# Ubuntu, CUDAを入れたあとにSteamが起動しない

- <https://qiita.com/yakitatata/items/b2203f90defcad68bb9f>

NVIDIAドライバのrunfileをダウンロードする（インストール済みのバージョンと合わせる）。

- ドライバ: <https://www.nvidia.co.jp/Download/Find.aspx?lang=jp>

ディレクトリに展開する。

```shell
chmod +x NVIDIA-Linux-x86_64-495.29.05.run
./NVIDIA-Linux-x86_64-495.29.05.run -x

sudo chown -R root:root NVIDIA-Linux-x86_64-495.29.05/
sudo mv NVIDIA-Linux-x86_64-495.29.05 /opt/
```

## /etc/ld.so.conf.d/nvidia-32bit.conf

32bitライブラリの入ったディレクトリをldに読み込ませるようにする。

```
/opt/NVIDIA-Linux-x86_64-495.29.05/32
```

再起動する。

```shell
sudo ldconfig

sudo reboot
```
