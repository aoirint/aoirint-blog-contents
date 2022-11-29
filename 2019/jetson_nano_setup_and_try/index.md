---
# moved from https://aoirint.hatenablog.com/entry/2019/12/02/100000
title: Jetson Nano セットアップとおためし
date: '2019-12-02 10:00:00'
draft: false
channel: 技術ノート
category: 'Get started'
tags:
  - 'Get started'
  - Jetson Nano
  - 電子工作
  - 機械学習
  - 組み込み
---
# Jetson Nano セットアップとおためし

メモ。

## TPLink Wireless Driver

[開発ボード](https://developer.nvidia.com/embedded/jetson-nano-developer-kit)にはWi-Fiモジュールが乗ってないので、USBドングルで無線LANにつなぐ（セットアップは有線で）。

[TPLink Archer T2U Nano](https://www.tp-link.com/jp/home-networking/adapter/archer-t2u-nano/)を使う。

- [tp-link Archer T2U Nano AC600をLinuxで使う方法 ｜ Shizuka's Style (Duo)](https://yaplog.jp/shizuka2/archive/256)
- [Jetson Nano で TP-LINK Archer T2U Nano を使う - Qiita](https://qiita.com/daisuzu_/items/8d6913f3bda1b7434526)
- [https://devtalk.nvidia.com/default/topic/1051503/jetson-nano/make-usb-wifi-dongle-rtl8812au-works-on-nano/](https://devtalk.nvidia.com/default/topic/1051503/jetson-nano/make-usb-wifi-dongle-rtl8812au-works-on-nano/)

NVIDIA Developer Forumsのつい最近（11/27）のポストで、新しいcommitで動かなくなっちゃったからCommit ID`d277c36`がいいよ、っていってる人がいる。でもT2U NanoのIDが登録されたのは2つ後の`4235b0e`なので、これを使ってみたけど、これで動いた（OSはr32.2.3）。この1つ後（現在masterの最新commit）の`fa68771`は接続が確立しなかった（パスワードの入力を何度でも求められる）。警告回避のためにリファクタリングしたらバグったっぽい（[コミットコメント](https://github.com/abperiasamy/rtl8812AU_8821AU_linux/commit/fa68771376a637c0b5f9cfa53da008570939a259#commitcomment-35946948)、[プルリク](https://github.com/abperiasamy/rtl8812AU_8821AU_linux/pull/310)）。

```shell
sudo apt install dkms

git clone https://github.com/abperiasamy/rtl8812AU_8821AU_linux.git
git checkout 4235b0ec7d7220a6364586d8e25b1e8cb99c36f1

Edit Makefile
CONFIG_PLATFORM_I386_PC = n
CONFIG_PLATFORM_ARM_JET_NANO = y

sudo make -f Makefile.dkms install
reboot
```

## ibus-mozc

```shell
sudo apt install ibus-mozc
killall ibus-daemon
ibus-daemon -x &
```

## Python

PyTorch使うならシステムのPython 3.6.9を使うのがよさそうだったので、そのまま

```shell
sudo apt install python3-pip
```

## jetson-stats

- [Jetson NanoのGPUモニタリング - Qiita](https://qiita.com/yamamo-to/items/161f7dcf96704b07a1f9)

いい感じに状態を確認できるやつ。

```shell
# pip3 install jetson-stats
# jtop
NVIDIA Jetson NANO/TX1 - Jetpack UNKNOWN [L4T 32.2.3]
CPU1 [|||||    Schedutil -  21%] 614MHz
CPU2 [|||      Schedutil -  15%] 614MHz
CPU3 [|||||    Schedutil -  20%] 614MHz
CPU4 [||||     Schedutil -  18%] 614MHz

Mem [|||||||||||||||||||||||||||||||                   2.1G/4.0GB] (lfb 95x4MB)
Imm [                                                    0kB/252kB] (lfb 252kB)
Swp [                                                 0.0GB/2.0GB] (cached 0MB)
EMC [||                                                              4%] 1.6GHz

GPU [||||                                                            7%] 153MHz
Dsk [########################                                    10.1GB/29.2GB]
          [info]          [Sensor]   [Temp]         [Power/mW]   [Cur]   [Avr]
UpT: 0 days 0:30:1        AO         39.50C         POM_5V_CPU   288     672
FAN [         0%] Ta=  0% CPU        32.00C         POM_5V_GPU   82      85
Jetson Clocks: [inactive] GPU        28.00C         POM_5V_IN    1895    2511
NV Power[0]: MAXN         PLL        30.00C
APE: 25MHz                PMIC      100.00C
HW engine:                thermal    29.75C
 ENC: NOT RUNNING
 DEC: NOT RUNNING
```

いちおうデフォルトでも`tegrastats`で状態をwatchできる。

## パフォーマンス最大化（発熱注意）

5V4AのDC電源を用意して、J48にジャンパーピンを挿す（電源不足で落ちるかも）。

コマンド`jetson_clocks`はオプションなしで呼び出すとCPU、GPU、EMC（メモリ？）のクロックを最大化する。熱暴走するかもなので熱対策してない場合はやらない方がいいかも。

```shell
ヘルプ
# jetson_clocks -h

デフォルトの設定
# jetson_clocks --show
SOC family:tegra210  Machine:NVIDIA Jetson Nano Developer Kit
Online CPUs: 0-3
CPU Cluster Switching: Disabled
cpu0: Online=1 Governor=schedutil MinFreq=102000 MaxFreq=1428000 CurrentFreq=1036800 IdleStates: WFI=1 c7=1 
cpu1: Online=1 Governor=schedutil MinFreq=102000 MaxFreq=1428000 CurrentFreq=921600 IdleStates: WFI=1 c7=1 
cpu2: Online=1 Governor=schedutil MinFreq=102000 MaxFreq=1428000 CurrentFreq=1132800 IdleStates: WFI=1 c7=1 
cpu3: Online=1 Governor=schedutil MinFreq=102000 MaxFreq=1428000 CurrentFreq=1428000 IdleStates: WFI=1 c7=1 
GPU MinFreq=76800000 MaxFreq=921600000 CurrentFreq=230400000
EMC MinFreq=204000000 MaxFreq=1600000000 CurrentFreq=1600000000 FreqOverride=0
Fan: speed=0
NV Power Mode: MAXN

# jetson_clocks --store ~/default_clocks.data

# jetson_clocks

# jetson_clocks --show
SOC family:tegra210  Machine:NVIDIA Jetson Nano Developer Kit
Online CPUs: 0-3
CPU Cluster Switching: Disabled
cpu0: Online=1 Governor=schedutil MinFreq=1428000 MaxFreq=1428000 CurrentFreq=1428000 IdleStates: WFI=0 c7=0 
cpu1: Online=1 Governor=schedutil MinFreq=1428000 MaxFreq=1428000 CurrentFreq=1428000 IdleStates: WFI=0 c7=0 
cpu2: Online=1 Governor=schedutil MinFreq=1428000 MaxFreq=1428000 CurrentFreq=1428000 IdleStates: WFI=0 c7=0 
cpu3: Online=1 Governor=schedutil MinFreq=1428000 MaxFreq=1428000 CurrentFreq=1428000 IdleStates: WFI=0 c7=0 
GPU MinFreq=921600000 MaxFreq=921600000 CurrentFreq=921600000
EMC MinFreq=204000000 MaxFreq=1600000000 CurrentFreq=1600000000 FreqOverride=1
Fan: speed=255
NV Power Mode: MAXN

# jetson_clocks --restore ~/default_clocks.data
# jetson_clocks --show
SOC family:tegra210  Machine:NVIDIA Jetson Nano Developer Kit
Online CPUs: 0-3
CPU Cluster Switching: Disabled
cpu0: Online=1 Governor=schedutil MinFreq=102000 MaxFreq=1428000 CurrentFreq=1428000 IdleStates: WFI=1 c7=1 
cpu1: Online=1 Governor=schedutil MinFreq=102000 MaxFreq=1428000 CurrentFreq=921600 IdleStates: WFI=1 c7=1 
cpu2: Online=1 Governor=schedutil MinFreq=102000 MaxFreq=1428000 CurrentFreq=1132800 IdleStates: WFI=1 c7=1 
cpu3: Online=1 Governor=schedutil MinFreq=102000 MaxFreq=1428000 CurrentFreq=1326000 IdleStates: WFI=1 c7=1 
GPU MinFreq=76800000 MaxFreq=921600000 CurrentFreq=230400000
EMC MinFreq=204000000 MaxFreq=1600000000 CurrentFreq=1600000000 FreqOverride=0
Fan: speed=0
NV Power Mode: MAXN
```

## PyTorch

- [https://devtalk.nvidia.com/default/topic/1049071/jetson-nano/pytorch-for-jetson-nano-version-1-3-0-now-available/](https://devtalk.nvidia.com/default/topic/1049071/jetson-nano/pytorch-for-jetson-nano-version-1-3-0-now-available/)

```shell
pip3 install numpy # takes long..
pip3 install torch-1.3.0-cp36-cp36m-linux_aarch64.whl
```

## TorchVision

setup.pyからinstallの途中でPillowが自動でビルドされるけど、ライブラリがないと失敗するので事前インストール。

必須は`libjpeg`と`zlib`。あと使いそうな`libfreetype`も（`zlib`と`libfreetype`はデフォルトで入ってる）。

- [Installation — Pillow (PIL Fork) 6.2.1 documentation](https://pillow.readthedocs.io/en/stable/installation.html)

```shell
sudo apt install libjpeg-turbo8-dev zlib1g-dev
sudo apt install libfreetype6-dev

pip3 install Pillow
```

```shell
git clone https://github.com/pytorch/vision.git
cd vision
python3 setup.py install --user # takes long..
```

## 動作テスト

（Clock：デフォルト）

```shell
pip3 install ipython
echo 'export PATH=$HOME/.local/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

```python
In [1]: import torch                                                            

In [2]: torch.cuda.is_available()
Out[2]: True

In [3]: torch.cuda.device_count()
Out[3]: 1

In [4]: device = torch.device('cuda:0')

In [5]: x = torch.randn(1, 3, 40, 40)

In [6]: x = x.to(device)

In [7]: x.shape
Out[7]: torch.Size([1, 3, 40, 40])

In [8]: x
Out[8]: 
tensor([[[[-0.0362, -0.6796,  0.6484,  ..., -0.5588,  0.3949, -0.8214],
          [-0.5975,  0.6352,  0.9712,  ..., -0.4434,  0.4439, -0.2363],
          [ 0.2326,  0.1052,  0.5346,  ..., -0.7047, -0.0173, -1.1312],
          ...,
          [-0.0730, -0.7666,  0.7147,  ...,  0.8821, -0.0262, -0.6976],
          [ 0.9593,  1.0495,  0.6041,  ...,  0.2833,  0.5237, -0.5829],
          [-0.4293, -1.9287,  0.6741,  ..., -1.0791, -0.5570, -0.3463]],

         [[-0.5106, -0.3363,  0.2770,  ..., -1.6835,  2.2409,  0.5745],
          [-0.9233,  0.7389,  0.6966,  ..., -0.5657, -0.4024, -0.5671],
          [-1.7160, -1.3546,  0.4675,  ...,  1.8385, -0.9948,  1.0485],
          ...,
          [ 0.8693,  0.2434,  0.7501,  ..., -0.9752,  2.3783, -0.4887],
          [-0.2279, -1.1861, -1.2003,  ..., -0.7934,  0.3169, -2.3324],
          [-1.1039, -1.6662, -0.0719,  ...,  0.6115,  2.2238, -0.8375]],

         [[-1.2368,  0.3786, -0.7985,  ...,  0.0504,  0.3354, -0.3505],
          [ 0.3309,  0.8257,  0.4800,  ..., -1.6101, -0.4429,  0.3643],
          [ 0.5604,  1.6997, -0.5299,  ..., -0.4896,  1.0926,  0.0423],
          ...,
          [ 1.8049, -1.0500, -1.3723,  ..., -0.4516,  0.5884,  1.3404],
          [ 0.3622,  0.6343, -0.0500,  ..., -0.0991,  0.9009,  0.7298],
          [ 2.3776, -0.1111,  0.2054,  ..., -1.4465, -2.2340, -1.0085]]]],
       device='cuda:0')
```

動いたけど、toがめっちゃ遅かった気がする..

## toが遅い。なんぞこれ

```python
In [1]: import time
In [2]: import torch

In [3]: x = torch.randn(1, 3, 40, 40)

In [4]: ts = time.time(); x = x.cuda(); elapsed = time.time() - ts;

In [5]: elapsed
Out[5]: 8.121735572814941
```

8秒...

```python
In [6]: z = torch.randn(1, 3, 40, 40)

In [7]: ts = time.time(); z = z.cuda(); elapsed = time.time() - ts;

In [8]: elapsed
Out[8]: 0.0011272430419921875

In [9]: y = torch.randn(1, 3, 512, 512)

In [10]: ts = time.time(); y = y.cuda(); elapsed = time.time() - ts;

In [11]: elapsed
Out[11]: 0.02282571792602539
```

初回だけ遅い。初期化でも走ってるんだろうか..

ipythonプロセスのMemを見てみると、初回のtoGPUの前後で600MiBくらいMem使用量が増えたが、2回目以降は0.1MiBくらいの増加だった。初回になにかしらの初期化処理が入ってるっぽい..

## メモリ

デスクトップ環境あり、ブラウザ付きでMem 2.2/4.0GB、Swap 1.3/2.0GB。

```python
import torch

x = torch.randn(1, 3, 512, 512)
x = x.cuda()

z = torch.randn(1, 3, 512, 512)
z = z.cuda()
```

ipythonプロセスをみると、起動直後33.6MiB、`import torch`で69.0MiB、`x = torch.randn(1, 3, 512, 512)`で72.4MiB、`x = x.cuda()`で726MiB、`z = torch.randn(1, 3, 512, 512)`で729MiB、`z = z.cuda()`で729Mib変化なし。

## 速度

（Clock：デフォルト）

```python
In [1]: import time

In [2]: import torch

In [3]: import torchvision.models as M

In [4]: x = torch.randn(1, 3, 224, 224)

In [5]: x = x.cuda()

In [6]: model = M.resnet50(pretrained=False)

In [7]: model = model.cuda()

In [8]: model.eval();

In [9]: with torch.no_grad():
   ...:     ts = time.time()
   ...:     y = model(x)
   ...:     elapsed = time.time() - ts
   ...:                                                                         

In [10]: elapsed                                                                
Out[10]: 20.638437747955322

In [11]: y.device                                                               
Out[11]: device(type='cuda', index=0)
```

あれ？ とても遅い。

```python
In [12]: z = torch.randn(1, 3, 224, 224)                                        

In [13]: z = z.cuda()                                                           

In [14]: with torch.no_grad(): 
    ...:     ts = time.time() 
    ...:     w = model(z) 
    ...:     elapsed = time.time() - ts 
    ...:                                                                        

In [15]: elapsed                                                                
Out[15]: 0.5333945751190186
```

うーん？

```python
In [16]: s = torch.randn(1, 3, 224, 224)

In [17]: s = s.cuda()

In [18]: with torch.no_grad():
    ...:     ts = time.time()
    ...:     t = model(s)
    ...:     elapsed = time.time() - ts
    ...:     

In [19]: elapsed
Out[19]: 0.04768824577331543

In [20]: u = torch.randn(1, 3, 224, 224).cuda()

In [21]: with torch.no_grad():
    ...:     ts = time.time()
    ...:     v = model(u)
    ...:     elapsed = time.time() - ts
    ...:     

In [22]: elapsed
Out[22]: 0.045920610427856445
```

2回くらい実行しないといいパフォーマンスがでないみたいですね..

仕切り直しでもう一回。

```python
In [1]: import time

In [2]: import torch

In [3]: import torchvision.models as M

In [4]: model = M.resnet50(pretrained=False)

In [5]: model = model.cuda()

In [6]: model.eval();

In [7]: x = torch.randn(1, 3, 224, 224)

In [8]: x = x.cuda()

In [9]: with torch.no_grad():
   ...:     ts = time.time()
   ...:     y = model(x)
   ...:     elapsed = time.time() - ts
   ...:     

In [10]: elapsed                                                                
Out[10]: 7.917057514190674
```

## スワップ作成

- [AWS Amazon Linux スワップファイル作成によりSwap領域のサイズを増やす - Qiita](https://qiita.com/na0AaooQ/items/278a11ed905995bd16af)

このあとの節でメモリ不足に陥ったのでswapfileを追加する（Default Mem: 4GB、Swap: 2GB）。2GB追加することにする。Swapに乗るとパフォーマンス低下するはずなので注意。

次の節で変換をJetson Nanoでゴリ押さなければ必要ないかもしれない。

```shell
# check storage capacity
df -h

dd if=/dev/zero of=/swapfile bs=1M count=2048
chmod 600 /swapfile

mkswap /swapfile
swapon /swapfile

# Append to /etc/fstab
/swapfile            swap                     swap           defaults                                     0 0

# check swap size
jtop
```

## 高速化（TensorRT）

- [Jetson Nanoでリアルタイムに物体検出をする方法（TensorFlow Object Detection API/NVIDIA TensorRT） - Qiita](https://qiita.com/karaage0703/items/67050f2418aa6bb3851a#nvidia-tensorrt%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%9F%E3%83%A2%E3%83%87%E3%83%AB%E3%81%AE%E6%9C%80%E9%81%A9%E5%8C%96)

高速化のためにはNVIDIAのTensorRTなるプログラムでモデルを最適化する必要があるらしい。1回目2回目のオーバーヘッドがなくなることを期待。

- [NVIDIA TensorRT | NVIDIA Developer](https://developer.nvidia.com/tensorrt)

> You can import trained models from every deep learning framework into TensorRT.

- [GitHub - NVIDIA-AI-IOT/torch2trt: An easy to use PyTorch to TensorRT converter](https://github.com/NVIDIA-AI-IOT/torch2trt)

PyTorch用の変換スクリプトはこれっぽい。Pythonコード上でパラメータ読み込み済みのモデルインスタンスを渡して変換、戻ってくるオブジェクト（`torch2trt.TRTModule`）はPyTorchの`nn.Module`を継承してて、ふつうのModule（Model）と同じように`torch.load`、`MODULE.load_state_dict`、`torch.save`、`MODULE.state_dict()`を使ってパラメータ読み書きできるみたい。

変換まではふつうのGPUマシンでやらないと重い（いちおう変換はできる）。

```shell
git clone https://github.com/NVIDIA-AI-IOT/torch2trt.git
cd torch2trt
python3 setup.py install --user
```

```python
import torch
import torchvision.models as M
from torch2trt import torch2trt
from torch2trt import TRTModule

# On GPU Machine
torch_model = M.resnet50(pretrained=False).cuda()
dummy = torch.zeros(1, 3, 224, 224).cuda()

trt_model = torch2trt(torch_model, [ dummy ])
torch.save(trt_model.state_dict(), 'trt_model.pth')

# On Jetson Nano
trt_model = TRTModule()
trt_model.load_state_dict(torch.load('trt_model.pth'))
```

```python
In [5]: ts = time.time(); model.load_state_dict(torch.load('trt_model.pth')); el
   ...: apsed = time.time() - ts

In [6]: elapsed
Out[6]: 38.654945611953735

In [7]: ts = time.time(); model.load_state_dict(torch.load('trt_model.pth')); el
   ...: apsed = time.time() - ts

In [8]: elapsed
Out[8]: 5.191670894622803

In [9]: ts = time.time(); model.load_state_dict(torch.load('trt_model.pth')); el
    ...: apsed = time.time() - ts

In [10]: elapsed
Out[10]: 7.234739542007446
```

```python
In [18]: ts = time.time(); x = torch.randn(1, 3, 224, 224).cuda(); elapsed = tim
    ...: e.time() - ts                                                          

In [19]: elapsed                                                                
Out[19]: 0.3353462219238281

In [20]: ts = time.time(); x = torch.randn(1, 3, 224, 224).cuda(); elapsed = tim
    ...: e.time() - ts                                                          

In [21]: elapsed                                                                
Out[21]: 0.016706228256225586

In [22]: ts = time.time(); x = torch.randn(1, 3, 224, 224).cuda(); elapsed = tim
    ...: e.time() - ts                                                          

In [23]: elapsed                                                                
Out[23]: 0.015973806381225586
```

```python
In [24]: ts = time.time(); y = model(x); elapsed = time.time() - ts             

In [25]: elapsed                                                                
Out[25]: 0.659074068069458

In [26]: ts = time.time(); y = model(x); elapsed = time.time() - ts             

In [27]: elapsed                                                                
Out[27]: 0.0033349990844726562

In [28]: ts = time.time(); y = model(x); elapsed = time.time() - ts             

In [29]: elapsed                                                                
Out[29]: 0.0031561851501464844
```

inferenceはだいぶ速い。ただモデルロードがとても遅い...。それに、初回実行の遅延は軽減してるとはいえ残ってる。

## SDカード

これまでの速度にも関わっていそうなので、SDカードについて調べてみる。

[UHS（Ultra-High Speed）とは - IT用語辞典 e-Words](http://e-words.jp/w/UHS.html)

安物を選んだので、UHS-I Class1 32GBのSamsung製Micro SDカード（これ）を使っている。UHS Class3の場合最低アクセス速度30MB/sだが、UHS Class1では最低アクセス速度10MB/sらしい。SDカードを変えれば（読み書きのかかるところでは）見込みで3倍以上速くなる可能性はあるのかなぁ？
