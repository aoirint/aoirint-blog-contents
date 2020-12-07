---
canonical_url: ./
title: OBS Studio on Ubuntu + NVIDIA GPUでWindow Captureを動作させる
# og_image:
# twitter_card: summary_large_image
og_description: OBS Studio on Ubuntu + NVIDIA GPUでWindow Captureを動作させる
date: '2020-11-09 06:10:00'
draft: false
category: OBS Studio
tags:
  - 'OBS Studio'
  - 'Ubuntu'
---

# OBS Studio on Ubuntu + NVIDIA GPUでWindow Captureを動作させる

Ubuntu + NVIDIA GPU環境では、OBS StudioでWindow Captureしようとすると
一部のアプリケーションで黒画面の表示になり動作しない。
おそらくOpenGLで描画しているものが対象。

- [Bug Report - [SOLVED] Window Capture Black Screen | OBS Forums](https://obsproject.com/forum/threads/solved-window-capture-black-screen.47082/)

私の環境では
`Window Capture (Xcomposite)`の設定欄をスクロールして
`Use alpha-less texture format (Mesa workaround)`にチェックを入れることで
正しく表示されるようになった。

Ubuntu 18.04, X.Org X Server 1.19.6, GDM 3.28.3, NVIDIA Driver 440.33.01、OBS Studio 26.0.2 (64bit)で動作確認。
