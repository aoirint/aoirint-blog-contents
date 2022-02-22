---
# moved from https://aoirint.hatenablog.com/entry/2020/03/24/154739
title: HinaOutlineProcessor（ひなホップ） 2017/01-03
date: '2020-03-24 15:47:39'
draft: true
channel: 技術ノート
category: Work
tags:
- Work
---
# HinaOutlineProcessor（ひなホップ） 2017/01-03

2017年の頭に開発していたxmlベースの階層型アウトラインプロセッサ。Java Swing製。

[https://github.com/aoirint/HinaOutlineProcessor:embed]

[https://twitter.com/kanomiya/status/836616749252390912:embed]

UIの仕様はStory Editor（旧）をベースにしている。Story Editorのモダン化とマルチプラットフォーム化ができればいいな、と思っていた。マルチプラットフォームならWebでは、と思われるかもしれないが、ローカルにファイルを置いておきたい気持ち（また、同時編集の沼も存在する）。その後、先行ツールの意外な高機能性の発見や大学進学のストレスなどでモチベが失われ、Minecraft Moddingを再開した。

[f:id:aoirint:20200324150508j:plain][f:id:aoirint:20200324150504j:plain][f:id:aoirint:20200324150506j:plain]

UI概観。


[f:id:aoirint:20200324150455j:plain]

ドキュメントファイルはこんな感じ。これを作っていた時期に関しては見逃してほしい。

[f:id:aoirint:20200324150459j:plain][f:id:aoirint:20200324150501j:plain]

設定ファイル。

[https://youtu.be/WE1oR2c4lQs:embed]

SQLite BrowserのUIに刺激されて言語変更機能を付けた。
