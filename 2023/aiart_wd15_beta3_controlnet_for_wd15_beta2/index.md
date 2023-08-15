---
title: 'AIアート WD 1.5 Beta3 + ControlNet for WD 1.5 Beta2 (furusu/ControlNet)'
date: '2023-08-15T10:30:00+09:00'
draft: true
noindex: false
channel: 技術ノート
category: aiart
tags:
  - aiart
  - waifu_diffusion
---
# AIアート WD 1.5 Beta3 + ControlNet for WD 1.5 Beta2 (furusu/ControlNet)

WD 1.5 Beta2用に作られたControlNetモデルを使って、WD 1.5 Beta3でAIイラストを生成してみました。

- [ControlNetでポーズや構図を指定してAIイラストを生成する方法｜ddPn08｜note](https://note.com/ddpn08/n/n7fce838499e7)
- [Stable Diffusion 2.1用のControlNetのセットアップ方法](https://webbigdata.jp/post-18036/)

## 環境

### クライアント

- Windows 10
- Google Chrome 115
- [デザインドール](https://terawell.net/ja/) v5.7.0.1

### GPUサーバ

- Ubuntu 22.04
- NVIDIA GeForce GTX 1080 Ti (VRAM 11GB)
- Dockerイメージ `aoirint/sd_webui:20230731.1`
  - Docker Hub: [aoirint/sd_webui](https://hub.docker.com/layers/aoirint/sd_webui/20230731.1/images/sha256-18d9fe63e746636bfbcdf1340732324479ccb20ffae6b8d265730f29bcea23e5)
  - GitHub: [aoirint/stable-diffusion-webui-docker](https://github.com/aoirint/stable-diffusion-webui-docker/releases/tag/20230731.1)
  - Ubuntu 22.04
  - Python 3.10.12
  - CUDA 11.8
  - WebUI v1.5.1

## モデル

- Stable Diffusion: Waifu Diffusion 1.5 Beta3 Radiance Fp16
  - `wd-1-5-beta3-radiance-fp16.safetensors [SHA256: fe8d7785d6]`
  - Hugging Face: [waifu-diffusion/wd-1-5-beta3](https://huggingface.co/waifu-diffusion/wd-1-5-beta3/tree/0850b219a40a86df205121c5ed71348cac20abc3)
- VAE: kl-f8-anime2
  - `kl-f8-anime2.ckpt [SHA256: df3c506e51]`
  - Hugging Face: [hakurei/waifu-diffusion-v1-4: kl-f8-anime2.ckpt](https://huggingface.co/hakurei/waifu-diffusion-v1-4/blob/6b239e9a5f0cdeba45131cde0fade1753179da4f/vae/kl-f8-anime2.ckpt)

### furusu/ControlNet

furusu氏によって、Waifu Diffusion 1.5 Beta2向けのControlNetモデルが配布されています。

- [https://huggingface.co/furusu/ControlNet/tree/b94c1fe0e72c147969f0ece81400a9b7ccc43272](https://huggingface.co/furusu/ControlNet/tree/b94c1fe0e72c147969f0ece81400a9b7ccc43272)

現在、以下のようなモデルが含まれています。

- Canny (Diff)
- Depth (Diff)
- OpenPose (Diff)
- Edge Depth Pose
- Tile

## デザインドール

ControlNetでは、ポーズを作成するデジタルデッサン人形がほしくなります。

デザインドールは、商用・非商用問わず利用できる、デジタルデッサン人形の個人開発ソフトウェアです。
有料ライセンスが￥7980で販売されているほか、一部の機能が制限された体験版を無料で使うことができます。

v5.7以降、AI画像生成での需要に応えて、OpenPose形式のポーズ画像の出力に対応したようです（体験版でも利用可）。

## プロンプト

[WD 1.5 Beta3のリリースノート](https://saltacc.notion.site/saltacc/WD-1-5-Beta-3-Release-Notes-1e35a0ed1bb24c5b93ec79c45c217f63)におすすめプロンプトが載っているので使います。

### ポジティブプロンプト

```
(exceptional, best aesthetic, new, newest, best quality, masterpiece, extremely detailed, anime, waifu:1.2)
```

### ネガティブプロンプト

```
lowres, ((bad anatomy)), ((bad hands)), missing finger, extra digits, fewer digits, blurry, ((mutated hands and fingers)), (poorly drawn face), ((mutation)), ((deformed face)), (ugly), ((bad proportions)), ((extra limbs)), extra face, (double head), (extra head), ((extra feet)), monster, logo, cropped, worst quality, jpeg, humpbacked, long body, long neck, ((jpeg artifacts)), deleted, old, oldest, ((censored)), ((bad aesthetic)), (mosaic censoring, bar censor, blur censor)
```
