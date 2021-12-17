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

コンテキスト共有のために、ググりながら背景など記述してみますが、**正確でない表現が含まれている可能性があります。**

## VOICEVOXとは

VOICEVOXは、ヒホ（ヒロシバ）氏が2021年8月にリリースした無料の音声合成ソフトウェアです。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉<br />無料で使える中品質なテキスト音声合成ソフトウェア、<a href="https://twitter.com/hashtag/VOICEVOX?src=hash&amp;ref_src=twsrc%5Etfw">#VOICEVOX</a> をリリースしました<br />🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉<br /><br />ぜひダウンロードして遊んでみてください！<a href="https://t.co/6MMth631mf">https://t.co/6MMth631mf</a></p>&mdash; ヒホ（ヒロシバ）🗑️ (@hiho_karuta) <a href="https://twitter.com/hiho_karuta/status/1421485814400184323?ref_src=twsrc%5Etfw">July 31, 2021</a></blockquote>

キャラクターイメージと音声ライブラリをセットにして提供する、キャラクター音声合成（話声合成）を主なターゲットとしているようです。

キャラクター音声合成を提供する既存のソフトウェア・サービスには、VOICEROID、CeVIO、A.I. VOICE、CoeFontなどがあります。
<!-- また、汎用の音声合成エンジン（AquesTalkなど）を活用してキャラクターに声を当てる「ゆっくり実況」のような文化があります。 -->

これらのソフトウェア・サービスはよく、ゲーム実況やキャラクター劇場動画などに使われています。

VOICEVOX 0.9現在、4人のキャラクター「四国めたん」「ずんだもん」「春日部つむぎ」「波音リツ」が利用可能です。

また「四国めたん」「ずんだもん」は、4つのスタイル（声色、声質）「ノーマル」「あまあま」「ツンツン」「セクシー」が利用可能です（「ノーマル」は非公式名称）。

![製品版VOICEVOX 0.9.3のスクリーンショット](images/voicevox_propr.png)

## 製品版VOICEVOXとOSS版VOICEVOX

VOICEVOXの開発では、開発の便宜上、製品版VOICEVOX、OSS版VOICEVOXという呼称が使われることがあります。

これらの詳細については、リポジトリに載せられているドキュメントを参照するのがいいと思います。

- [VOICEVOXのOSSコミュニティ](https://github.com/VOICEVOX/voicevox/blob/52a01ccba5e2c627298b8661e2df004f410b5594/public/ossCommunityInfos.md)
- [VOICEVOXの全体構成](https://github.com/VOICEVOX/voicevox/blob/52a01ccba5e2c627298b8661e2df004f410b5594/docs/%E5%85%A8%E4%BD%93%E6%A7%8B%E6%88%90.md)

## コントリビュートの経緯

自分のモチベーションを把握しておきたいので、経緯を書いてみます。

8月1日の朝9時ごろ、東北ずん子公式さんのツイートがTLに流れてきました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">こちらが、めたんちゃん、ずんだもんの音声合成ソフトVOICEVOXになりますo(≧▽≦)o<a href="https://t.co/khcYXoN0pR">https://t.co/khcYXoN0pR</a> <a href="https://t.co/tb09I4K5dI">pic.twitter.com/tb09I4K5dI</a></p>&mdash; 東北ずん子(公式)💚12月23日26時25分から東北放送でTV放送 (@t_zunko) <a href="https://twitter.com/t_zunko/status/1421485817319546884?ref_src=twsrc%5Etfw">July 31, 2021</a></blockquote>

無料で使えるいい感じの品質の音声合成ソフトがあるらしいぞ、ということで、これをきっかけにして、ふだん自分が使っている環境のLinux（Ubuntu）上での動作を試み始めました（0.6まではWindowsバイナリのみの提供）。

もともと個人的に、Windowsバイナリしかない研究用ライブラリをLinux上で動かすために、
[Wine in Docker環境を作成していた](https://twitter.com/aoirint/status/1381667075341447169)ので、
それを改変してVOICEVOXソフトウェアを動かしてみました。

<blockquote class="twitter-tweet"><p lang="und" dir="ltr"><a href="https://t.co/WuOGw8LUTj">pic.twitter.com/WuOGw8LUTj</a></p>&mdash; aoi🌱 (@aoirint) <a href="https://twitter.com/aoirint/status/1424142369046155268?ref_src=twsrc%5Etfw">August 7, 2021</a></blockquote>

初期のバージョン（上の動画は0.1.1）ではちょっと音声がかすれ気味だったり、CPU版での合成に長めの時間がかかったりしていたのですが、バージョンが上がることで品質や合成速度にも調整が加えられています。

0.5まで、製品版音声ライブラリは、製品版ソフトウェアに同梱されたWindows向け（DLL）のみ存在していたため、Wineや仮想環境を介さずネイティブ動作させることはできませんでした。

[0.5.2](https://github.com/VOICEVOX/voicevox_core/releases/tag/0.5.2)で、Linux向けライブラリ（SO）を含む音声ライブラリ単体の提供が始まったので、その後のバージョンでソフトウェアのLinuxネイティブ対応や自動ビルドができるようになりました。

## VOICEVOXの構成と開発

VOICEVOXソフトウェアは、いまのところ、
Electron + Vuex フロントエンド（エディター） [VOICEVOX](https://github.com/VOICEVOX/voicevox)、
FastAPI HTTP 音声合成サーバ [VOICEVOX ENGINE](https://github.com/VOICEVOX/voicevox_engine)、
音声合成の計算をするライブラリ [VOICEVOX CORE](https://github.com/VOICEVOX/voicevox_core)の3つから構成されています。

- [VOICEVOXの構成](https://github.com/VOICEVOX/voicevox/blob/52a01ccba5e2c627298b8661e2df004f410b5594/docs/%E5%85%A8%E4%BD%93%E6%A7%8B%E6%88%90.md#%E6%A7%8B%E6%88%90)

このうち、
[VOICEVOX エディター](https://github.com/VOICEVOX/voicevox)、
[VOICEVOX ENGINE](https://github.com/VOICEVOX/voicevox_engine)がOSS化され、コントリビューションの受け付けが行われています。

[VOICEVOX CORE](https://github.com/VOICEVOX/voicevox_core)は、0.9現在ビルド済みバイナリのみを提供する形ですが、
主要な実装のOSS化が計画されていて、計算ライブラリのLibTorch（TorchScript）からONNX Runtimeへの移行と合わせて進められています。

VOICEVOXの開発は、[ヒホ氏による開発生放送](https://live.nicovideo.jp/watch/co3686550)（毎日23時ごろから2-3時間程度）、GitHub上のやり取り、コミュニティDiscordを主なコミュニケーション場所として進められています。

## VOICEVOX ENGINEのDockerイメージ化

まず、VOICEVOX ENGINE

## VOICEVOX ENGINEのLinux対応

## VOICEVOX ENGINEのビルド自動化

## VOICEVOX ソフトウェアのビルド自動化


## 今後

[0.8.0](https://github.com/VOICEVOX/voicevox_core/releases/tag/0.8.0)で、macOS向けライブラリ（dylib）の提供が始まったので、macOS対応が進められていて、0.10以降でのmacOS対応を見据えて、すでに動作する非公式ビルドが作成されています。
