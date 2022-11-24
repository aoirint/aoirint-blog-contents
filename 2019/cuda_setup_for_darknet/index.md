---
# moved from https://aoirint.hatenablog.com/entry/2019/09/27/170000
title: CUDA setup (make darknet)
date: '2019-09-27 17:00:00'
draft: false
channel: 技術ノート
category: Python
tags:
- Python
- 機械学習
---
# CUDA setup (make darknet)

darknetのmakeに失敗するので、CUDA/NVIDIA Driverの再セットアップ。

- [https://developer.nvidia.com/cuda-downloads](https://developer.nvidia.com/cuda-downloads)

`runfile (local)`をダウンロード。

```shell
apt purge nvidia-*
apt purge cuda-*
reboot
```

あとはrunfileを実行してCUIでNVIDIA DriverとCUDA Toolkitをインストール（CUIモードで`systemctl stop lightdm`←16.04の場合だった、18.04なら`gdm`？）。

```shell
export PATH=/usr/local/cuda/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
```

.bashrc（Ubuntuの場合）に上の2行を追記する。

- Ubuntu 18.04.3
- CUDA Toolkit 10.1 & NVIDIA Driver 418（cuda_10.1.243_418.87.00）
- darknet（[#61c9d02](https://github.com/pjreddie/darknet/tree/61c9d02ec461e30d55762ec7669d6a1d3c356fb2)）

上記の環境でdarknetのmakeに成功した。

[https://pjreddie.com/darknet/](https://pjreddie.com/darknet/)

## 参考

- [http://vastee.hatenablog.com/entry/2018/11/20/152234](http://vastee.hatenablog.com/entry/2018/11/20/152234)
