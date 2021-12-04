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

**このふわっとした技術共有記事を、お気持ち表明やコミュニティ・開発者等関係者への攻撃に利用するのはお控えください。**（面倒事を起こして迷惑はかけたくないのじゃ...）

**おねがいします😢**

## VOICEVOXとは

VOICEVOXは、ヒホ（ヒロシバ）氏が2021年8月にリリースした無料の音声合成ソフトウェアです。

[リリースツイート]()

VOICEVOXには、製品版VOICEVOXとOSS版VOICEVOXという2つのパッケージがあります。

製品版VOICEVOXは、話者の権利などで保護された音声ライブラリやイラストなどを同梱し、[VOICEVOX 公式サイト](https://voicevox.hiroshiba.jp/)で配布しているパッケージです。

通常のテキスト読み上げを目的とするユーザ（ソフトウェアトーク動画投稿者など）は、製品版VOICEVOXを使用しています。

![製品版VOICEVOXのスクリーンショット]()

OSS版VOICEVOXは、プロプライエタリリソースを含まない、OSS開発可能なリソースで構成されたパッケージです。

![OSS版VOICEVOXのスクリーンショット]()

例えば、シロワニさん氏が2021年11月にリリースした無料の音声合成ソフトウェアCOEIROINKは、いまのところ、UIにOSS版VOICEVOXをforkして活用され、音声ライブラリは独自のものを搭載されています。

- <https://coeiroink.com/>


## 製品版VOICEVOXのキャラクター

せっかく記事を書いたので、製品版のキャラクターを簡単に紹介してみます。



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

- <>

## 事前調査: Wine in Dockerによる製品版VOICEVOXの起動


## VOICEVOX ENGINEのDockerイメージ化

まず、VOICEVOX ENGINE

## VOICEVOX ENGINEのLinux対応

## VOICEVOX ENGINEのビルド自動化

## VOICEVOX ソフトウェアのビルド自動化

## VOICEVOX COREのビルド自動化

