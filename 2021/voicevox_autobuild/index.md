---
title: テキスト読み上げソフトVOICEVOXのビルドを自動化した
draft: true
date: '2021-12-15 7:00'
category: レポート
tags:
    - 'CI CD'
    - 'GitHub Actions'
    - 音声合成
---
# テキスト読み上げソフトVOICEVOXのビルドを自動化した

## ここを必ず読んでね

この記事は、1コントリビュータ、1ユーザのふわっとした感想文です。

**VOICEVOXの今後の活躍と、開発の一助になればと思い、記事を作ってみます。**

コンテキスト共有のために、ググりながら背景など記述してみますが、**正確でない表現が含まれている可能性が高いです。**

**このふわっとした技術共有記事を、お気持ち表明やコミュニティ・開発者等関係者への攻撃に利用するのはお控えください。**

**おねがいします😢**

## VOICEVOXとは

VOICEVOXは、ヒホ（ヒロシバ）氏が2021年8月にリリースした無料の音声合成ソフトウェアです。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉<br />無料で使える中品質なテキスト音声合成ソフトウェア、<a href="https://twitter.com/hashtag/VOICEVOX?src=hash&amp;ref_src=twsrc%5Etfw">#VOICEVOX</a> をリリースしました<br />🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉<br /><br />ぜひダウンロードして遊んでみてください！<a href="https://t.co/6MMth631mf">https://t.co/6MMth631mf</a></p>&mdash; ヒホ（ヒロシバ）🗑️ (@hiho_karuta) <a href="https://twitter.com/hiho_karuta/status/1421485814400184323?ref_src=twsrc%5Etfw">July 31, 2021</a></blockquote>

ゲーム実況やキャラクター劇場動画などに使われる、キャラクター音声合成（話声合成）を主なターゲットとしているようです。

キャラクター音声合成を提供する既存のソフトウェア・サービスには、VOICEROID、CeVIO、A.I. VOICE、CoeFontなどがあります。

VOICEVOX 0.9現在、4人のキャラクター「四国めたん」「ずんだもん」「春日部つむぎ」「波音リツ」が利用可能です。

また「四国めたん」「ずんだもん」は、4つのスタイル（声色）「ノーマル」「あまあま」「ツンツン」「セクシー」が利用可能です（「ノーマル」は非公式名称）。

## 製品版VOICEVOXとOSS版VOICEVOX

VOICEVOXには、開発の効率をよくするため、製品版VOICEVOXとOSS版VOICEVOXという2つのパッケージ形式があります。

製品版VOICEVOXは、話者の権利などで保護された音声ライブラリやイラストなどを同梱し、[VOICEVOX 公式サイト](https://voicevox.hiroshiba.jp/)で配布しているパッケージです。

通常のテキスト読み上げを目的とするユーザ（ソフトウェアトーク動画投稿者など）は、製品版VOICEVOXを使用しています（スクリーンショットは0.9.3）。

![製品版VOICEVOX 0.9.3のスクリーンショット](images/voicevox_propr.png)

OSS版VOICEVOXは、VOICEVOXのプロプライエタリ部分を含まない、OSS開発可能なリソースで構成されたパッケージです（スクリーンショットは FRONT [52a01cc](https://github.com/VOICEVOX/voicevox/tree/52a01ccba5e2c627298b8661e2df004f410b5594), ENGINE [a451258](https://github.com/VOICEVOX/voicevox_engine/tree/a451258c937113e4d244a643cbaea9d86b1fdcc1)）。

![OSS版VOICEVOX のスクリーンショット](images/voicevox_oss.png)


## 製品版VOICEVOXのキャラクター

せっかく記事を書いたので、製品版のキャラクターを簡単に紹介してみます。

## コントリビュートまでの経緯

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">こちらが、めたんちゃん、ずんだもんの音声合成ソフトVOICEVOXになりますo(≧▽≦)o<a href="https://t.co/khcYXoN0pR">https://t.co/khcYXoN0pR</a> <a href="https://t.co/tb09I4K5dI">pic.twitter.com/tb09I4K5dI</a></p>&mdash; 東北ずん子(公式)💚12月23日26時25分から東北放送でTV放送 (@t_zunko) <a href="https://twitter.com/t_zunko/status/1421485817319546884?ref_src=twsrc%5Etfw">July 31, 2021</a></blockquote>

8月1日の朝9時ごろ、東北ずん子公式さんのツイートがTLに流れてきました。

これをきっかけにして、Linux上での動作を試み始めました。

もともと、Windowsバイナリしかない研究用ライブラリをLinux上で動かすために、
[Wine in Docker環境を作成していた](https://twitter.com/aoirint/status/1381667075341447169)ので、
それを改変してVOICEVOXソフトウェアを動かしていました。

<blockquote class="twitter-tweet"><p lang="und" dir="ltr"><a href="https://t.co/yIUikQe3Bt">pic.twitter.com/yIUikQe3Bt</a></p>&mdash; aoi🌱 (@aoirint) <a href="https://twitter.com/aoirint/status/1424142339996459008?ref_src=twsrc%5Etfw">August 7, 2021</a></blockquote>


## VOICEVOXの構成と開発

**以下では、基本的にOSS版VOICEVOXについて言及しています。**

VOICEVOXは、いまのところ、
Electron + Vuex フロントエンド [VOICEVOX](https://github.com/VOICEVOX/voicevox)、
FastAPI HTTP 音声合成サーバ [VOICEVOX ENGINE](https://github.com/VOICEVOX/voicevox_engine)、
音声ライブラリのインタフェースとなるライブラリ [VOICEVOX CORE](https://github.com/VOICEVOX/voicevox_core)の3つから構成されています。

このうち、
[VOICEVOX フロントエンド](https://github.com/VOICEVOX/voicevox)、
[VOICEVOX ENGINE](https://github.com/VOICEVOX/voicevox_engine)がOSS化され、コントリビューションの受け付けが行われています。

また[VOICEVOX CORE](https://github.com/VOICEVOX/voicevox_core)も、主要な実装のOSS化を計画しています。

VOICEVOXは、[ヒホ氏による生放送](https://live.nicovideo.jp/watch/co3686550)（毎日23時ごろから3時間程度）、コミュニティDiscord、GitHub上のやり取り

## 事前調査: Wine in Dockerによる製品版VOICEVOXの起動


## VOICEVOX ENGINEのDockerイメージ化

まず、VOICEVOX ENGINE

## VOICEVOX ENGINEのLinux対応

## VOICEVOX ENGINEのビルド自動化

## VOICEVOX ソフトウェアのビルド自動化

## VOICEVOX COREのビルド自動化

