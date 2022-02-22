---
# moved from https://aoirint.hatenablog.com/entry/2020/05/09/073722
title: Zoom API (OAuth)を試したメモ
date: '2020-05-09 07:37:22'
draft: false
channel: 技術ノート
category: Remote
tags:
- Remote
---
# Zoom API (OAuth)を試したメモ

お試しのためのOAuthアクセストークン（一時）を得るサンプルをクローン。

```sh
git clone https://github.com/zoom/zoom-oauth-sample-app.git
cd zoom-oauth-sample-app

npm install
```

https://github.com/zoom/zoom-oauth-sample-app

ngrok（remote.itみたいなやつ）をダウンロード（実行ファイル）。ngrok.ioのサブドメインにポートを転送する。

https://ngrok.com/

```sh
./ngrok http 4000
```

OAuth AppをZoom上に作成。

https://marketplace.zoom.us/develop/create


設定を書く.envファイルを作成。

```sh
touch .env
```

```
clientID=Zoom OAuth AppのClient ID
clientSecret=Zoom OAuth AppのClient Secret
redirectURL=ngrokのURL
```

起動。

```sh
npm run start
```

API Document: https://marketplace.zoom.us/docs/api-reference/introduction

上のURLでテスト呼び出しができるので、アクセストークンをとりあえず使いまわして検証。

管理者でない場合は得られる情報が結構限られる（進行中のミーティングの参加者リストは管理者でないととれなさそう）。管理者の場合はOAuthではなくJWTを使ったほうが良さそう。今回の目的は達成できなさそうだったのでこれで終了（トークンとらなくても入出力サンプルはある）。

JWT参考：インタラクション2020のリモート開催で使われたシステム https://github.com/hasevr/i2020zoom
