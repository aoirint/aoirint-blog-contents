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

こちらの記事は、[GitHub Actions Advent Calendar 2021](https://qiita.com/advent-calendar/2021/github-actions)の17日目の記事となります。

実用的な用例などのある記事ではないかもですが、GitHub・GitHub Actionsに大変お世話になっているので、個人的に感謝をこめて。

## ここを必ず読んでね

こちらの記事は、1コントリビュータ、1ユーザのふわっとした感想文で、勝手に書いた体験記になります。

**VOICEVOXの今後の活躍と、開発の一助になればと思い、記事として作ってみました。**

コンテキスト共有のために、ググりながら背景など記述してみますが、**正確でない表現が含まれている可能性があります**ので、ご注意くださいませ。

## 背景

### VOICEVOXとは

VOICEVOXは、ヒホ（ヒロシバ）氏が2021年8月にリリースした無料の音声合成ソフトウェアです。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉<br />無料で使える中品質なテキスト音声合成ソフトウェア、<a href="https://twitter.com/hashtag/VOICEVOX?src=hash&amp;ref_src=twsrc%5Etfw">#VOICEVOX</a> をリリースしました<br />🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉🎉<br /><br />ぜひダウンロードして遊んでみてください！<a href="https://t.co/6MMth631mf">https://t.co/6MMth631mf</a></p>&mdash; ヒホ（ヒロシバ）🗑️ (@hiho_karuta) <a href="https://twitter.com/hiho_karuta/status/1421485814400184323?ref_src=twsrc%5Etfw">July 31, 2021</a></blockquote>

キャラクターイメージと音声ライブラリをセットにして提供する、キャラクター音声合成（話声合成）を主なターゲットとしているようです。

キャラクター音声合成を提供する既存のソフトウェア・サービスには、VOICEROID、CeVIO、A.I. VOICE、CoeFontなどがあります。
<!-- また、汎用の音声合成エンジン（AquesTalkなど）を活用してキャラクターに声を当てる「ゆっくり実況」のような文化があります。 -->

これらのソフトウェア・サービスはよく、ゲーム実況やキャラクター劇場動画などに使われています。

VOICEVOX 0.9現在、4人のキャラクター「四国めたん」「ずんだもん」「春日部つむぎ」「波音リツ」が利用可能です。

また「四国めたん」「ずんだもん」は、4つのスタイル（声色、声質）「ノーマル」「あまあま」「ツンツン」「セクシー」が利用可能です（「ノーマル」は非公式名称）。

![製品版VOICEVOX 0.9.3のスクリーンショット](images/voicevox_propr.png)

### 製品版VOICEVOXとOSS版VOICEVOX

VOICEVOXの開発では、開発の便宜上、製品版VOICEVOX、OSS版VOICEVOXという呼称が使われることがあります。

これらの詳細については、リポジトリに載せられているドキュメントを参照するのがいいと思います。

- [VOICEVOXのOSSコミュニティ](https://github.com/VOICEVOX/voicevox/blob/52a01ccba5e2c627298b8661e2df004f410b5594/public/ossCommunityInfos.md)
- [VOICEVOXの全体構成](https://github.com/VOICEVOX/voicevox/blob/52a01ccba5e2c627298b8661e2df004f410b5594/docs/%E5%85%A8%E4%BD%93%E6%A7%8B%E6%88%90.md)

### モチベーション

自分のモチベーションを把握しておきたいので、個人的なことになりますが、経緯を書いてみます。

- 実質的な音声知識はほぼない
    - OpenJTalkのTTS機能を使っていた
    - 2019年5月頃に音声認識どうやったらできるのかスケッチを描いていた
        - 既存手法もまったく知らないので、むずかしそうだなぁということがわかった
    - 2019年6月頃にMLP 音声認識を積んでいた
        - 既存手法を知りたかった
    - 2021年5月頃に音声分析合成を積んでいた
        - CoeFont（4月）と七声ニーナ（5月）が背景
    - 2021年6月頃にA.I.VOICE 琴葉葵・茜を購入
- 深層学習
    - 流行ったときに画像系でChainer・PyTorchでちょっとした論文実装を書いたり書かなかったり
- 開発
    - Java、ちょっとだけPHP経験を背景
    - Web、ちょっとだけモバイル
    - Python、ちょっとだけTypeScriptが書ける
    - Dockerイメージをよく作っている

音声合成界隈には、VOICEROID・ソフトウェアトーク動画などを通じて興味を持っていました。

8月1日の朝9時ごろ、東北ずん子公式さんのツイートがTLに流れてきました。

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">こちらが、めたんちゃん、ずんだもんの音声合成ソフトVOICEVOXになりますo(≧▽≦)o<a href="https://t.co/khcYXoN0pR">https://t.co/khcYXoN0pR</a> <a href="https://t.co/tb09I4K5dI">pic.twitter.com/tb09I4K5dI</a></p>&mdash; 東北ずん子(公式)💚12月23日26時25分から東北放送でTV放送 (@t_zunko) <a href="https://twitter.com/t_zunko/status/1421485817319546884?ref_src=twsrc%5Etfw">July 31, 2021</a></blockquote>

無料で使えるいい感じの品質の音声合成ソフトがあるらしいぞ、ということで、これをきっかけにして、ふだん自分が使っている環境のLinux（Ubuntu）上での動作を試み始めました。

この時点ではWindowsバイナリのみの提供でしたが、もともと個人的に、他のWindowsバイナリしかない研究用ライブラリをLinux上で動かすために、
[Wine in Docker環境を作成していた](https://twitter.com/aoirint/status/1381667075341447169)ので、
それを改変してVOICEVOXソフトウェアを動かしてみました。

<blockquote class="twitter-tweet"><p lang="und" dir="ltr"><a href="https://t.co/WuOGw8LUTj">pic.twitter.com/WuOGw8LUTj</a></p>&mdash; aoi🌱 (@aoirint) <a href="https://twitter.com/aoirint/status/1424142369046155268?ref_src=twsrc%5Etfw">August 7, 2021</a></blockquote>

初期のバージョン（上の動画は0.1.1）では、ちょっと音声がかすれ気味だったり、CPU版での合成に長めの時間がかかったりしていました。
これについては、バージョンが上がることで音声品質や合成速度にも調整が加えられています。

0.5まで、製品版音声ライブラリは、製品版ソフトウェアに同梱されたWindows向け（DLL）のみ存在していたため、Wineや仮想環境を介さずネイティブ動作させることはできませんでした。

[0.5.2](https://github.com/VOICEVOX/voicevox_core/releases/tag/0.5.2)で、Linux向けライブラリ（SO）を含む音声ライブラリ単体の提供が始まったので、その後のバージョンでソフトウェアのLinuxネイティブ対応や自動ビルドができるようになりました。

### VOICEVOXの構成と開発

VOICEVOXソフトウェアは、いまのところ、
Electron + Vuex フロントエンド（エディター） [VOICEVOX](https://github.com/VOICEVOX/voicevox)、
FastAPI HTTP 音声合成サーバ [VOICEVOX ENGINE](https://github.com/VOICEVOX/voicevox_engine)、
音声合成の計算をするコアライブラリ [VOICEVOX CORE](https://github.com/VOICEVOX/voicevox_core)の3つから構成されています。

- [VOICEVOXの構成](https://github.com/VOICEVOX/voicevox/blob/52a01ccba5e2c627298b8661e2df004f410b5594/docs/%E5%85%A8%E4%BD%93%E6%A7%8B%E6%88%90.md#%E6%A7%8B%E6%88%90)

[VOICEVOX CORE](https://github.com/VOICEVOX/voicevox_core)は、0.9現在OSS開発の準備が完了しておらず、
[推論実装を別リポジトリで公開](https://github.com/Hiroshiba/vv_core_inference/tree/539ca8f90de038471d22857ffdff496db2788009)した上で、ビルド済みバイナリのみを提供する形になっていますが、
権利保護上必要な部分を除いた主要な実装のOSS化が計画されていて、計算ライブラリのLibTorch（TorchScript）からONNX Runtimeへの移行と合わせて進められています。

VOICEVOXの開発は、[GitHub](https://github.com/VOICEVOX)、[ヒホ氏による開発生放送](https://live.nicovideo.jp/watch/co3686550)（毎日23時ごろから数時間程度）、[コミュニティDiscord](https://twitter.com/hk_coil424/status/1432351677026160641)を主なコミュニケーション場所として進められています。


## 本題：テキスト読み上げソフトVOICEVOXのビルドを自動化した

大きなお題目を打ってはいますが、実際には、いろいろな人の力が合わさって進められています。

たしかにWindows・Linux向けバイナリの自動ビルドについて、主要部分の初期実装をしましたが、
コアライブラリの自動ビルドや他OS対応はヒホ氏が実装していますし、
バイナリ互換性などの面でもいろいろな人の助けを借りていて、
またメンテナンス・改良が日々行われています。

本格的なOSS貢献はVOICEVOXが初めてでしたが、手探りの中、温かいやり取りで非常にいい体験をすることができて、ありがたく思っています。

というわけで、どういう風にビルド自動化を進めていったかを書いていきます。

### VOICEVOX ENGINEのDockerイメージ化

まず、VOICEVOX ENGINEのDockerイメージの作成を始めました。

- <https://github.com/VOICEVOX/voicevox_engine/pull/86>

OSSにプルリクエストを送った経験がほとんどなかったので、まずは小さいパッチを送って様子を見たいと思いました。

- <https://github.com/miyadaiku/miyadaiku/pull/43>
- <https://github.com/miyadaiku/miyadaiku/pull/44>

身内以外へのプルリクエストは、上のMiyadaikuというPython製静的サイトジェネレータが初めてでした。

前々から追加したい機能があって、fork・改造して使っていたのですが、
本家にアップデートがあるたびにマージコミットを作ったりrebaseしたり、
gitリポジトリからパッケージをインストールしたりが面倒だったので、
取り込んでもらえないか試してみることにしました。

似た機能の要望Issueが自分の望む方法とは別の方法で実装されていて、
それでは目的を達成できませんでした。

いきなり機能追加提案やパッチを送っても、受け入れられるかよくわかりませんでした。

そもそも機能提案やプルリクエストを受け付けているのか、
開発上どういうルールがあるのか（開発の進め方とか、方針とか、コードフォーマットとか）、
メンテナの人となり、

メンテナにとっても、誰かよくわからない人が送ってきたパッチよりも、
過去にいい貢献をしてくれた人が送ってきたパッチのほうがポジティブに見ることができるかなと勝手に思っています。
機能提案があったとき、それがプロジェクトの方針に合っているか判断する必要があります。

相手が勘違いしていて的外れなことを言っている場合もあれば、
意図をうまく伝えられず、勘違いされてしまう場合もあると思っています。

実装や方針、プロジェクトで大切にしていることを読み取っておかなければ、ただ迷惑をかけたり、手間をかけさせたりしただけになってしまいます。

小さなパッチがRejectされても、


自分が使う上でバグを見つけていたので、まず小さいバグ修正パッチを送り、それから機能追加提案にあたるパッチを送りました。
変更内容が小さかったので、機能提案ついでにパッチを作りましたが、Issueを立てて実装方針を確認・議論することがだいたい望ましいと思います。

一度軽くコミュニケーションをとって、お互いの雰囲気や実装方針を読み取るために、まず小さなパッチを送ってみました。


大きなパッチを苦労して作っても、


開発中に、


### VOICEVOX ENGINEのLinux対応

### VOICEVOX ENGINEのビルド自動化

### VOICEVOX エディタのビルド自動化


## その他

[0.8.0](https://github.com/VOICEVOX/voicevox_core/releases/tag/0.8.0)で、macOS向けライブラリ（dylib）の提供が始まったので、macOS対応が進められていて、0.10以降でのmacOS対応を見据えて、すでに動作する非公式ビルドが作成されています。
