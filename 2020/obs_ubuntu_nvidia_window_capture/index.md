---
title: OBS Studio on Ubuntu 20.04 + NVIDIA GPUでWindow Captureを動作させる
date: '2020-11-09 06:10:00'
updated: '2022-01-23 11:50:00'
draft: false
category: OBS Studio
tags:
  - 'OBS Studio'
  - 'Ubuntu'
---

# OBS Studio on Ubuntu 20.04 + NVIDIA GPUでWindow Captureを動作させる

- [Bug Report - [SOLVED] Window Capture Black Screen | OBS Forums](https://obsproject.com/forum/threads/solved-window-capture-black-screen.47082/)

Ubuntu + NVIDIA GPU環境では、OBS StudioでWindow Captureしようとすると
一部のアプリケーションで黒画面の表示になり動作しない。
おそらくOpenGLで描画しているものが対象。

いくつか対処法の候補がある。

- Alt+F2から`r`を入力して実行
- アプリケーションをフルスクリーン表示からウインドウ表示に切り替える
- アプリケーションをウインドウ表示からフルスクリーン表示に切り替える
- `Window Capture (Xcomposite)`の設定欄をスクロールして、`Use alpha-less texture format (Mesa workaround)`にチェックを入れる
- OSを再起動（GPUドライバに更新が入ったときなど）
