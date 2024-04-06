---
title: 'さくらのVPS 10周年記念キャンペーン 第2弾 スタートアップスクリプトを自作して公開してみませんか！ キャンペーン'
date: '2021-03-20T13:11:27+09:00'
draft: false
channel: 技術ノート
category: 技術イベント
tags:
  - 技術イベント
  - さくらインターネット
  - さくらのVPS
---
# さくらのVPS 10周年記念キャンペーン 第2弾 スタートアップスクリプトを自作して公開してみませんか！ キャンペーン

GitHub Self Hosted RunnerをさくらのVPS上に構築するスタートアップスクリプトを投稿しました。

- [https://secure.sakura.ad.jp/vps-beta/startupscripts/e3ad9634-6cda-4eeb-8a7a-6f68efedcc58](https://secure.sakura.ad.jp/vps-beta/startupscripts/e3ad9634-6cda-4eeb-8a7a-6f68efedcc58)
- [https://github.com/aoirint/sakura_vps_github_runner](https://github.com/aoirint/sakura_vps_github_runner)
- [https://docs.github.com/en/free-pro-team@latest/actions/hosting-your-own-runners](https://docs.github.com/en/free-pro-team@latest/actions/hosting-your-own-runners)

さくらインターネットさんのさくらのVPSは2010年9月にサービス開始し、2020年9月で10周年だそうで、
記念キャンペーンをやっています（応募したのは第2弾ですが、第3弾もやっていました）。

- [https://vps.sakura.ad.jp/news/vps-10th-campaign2/](https://vps.sakura.ad.jp/news/vps-10th-campaign2/)

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">さくらのVPS 10周年キャンペーン 第2弾 開催のお知らせ🎉<a href="https://t.co/LQwYf0QjgS">https://t.co/LQwYf0QjgS</a></p>&mdash; さくらのVPS (@sakura_vps) <a href="https://twitter.com/sakura_vps/status/1333641110493687808?ref_src=twsrc%5Etfw">December 1, 2020</a></blockquote>

Twitterでこの10周年記念キャンペーンを見かけて、
ニンテンドープリペイドカードが当たる、ということで
~~さくらのVPSの費用回収をするために~~
Switchのゲームを買うために応募してみました（応募者が少なかったらしく当選した）。

<blockquote class="twitter-tweet"><p lang="zxx" dir="ltr"><a href="https://t.co/3kM9W53OCc">pic.twitter.com/3kM9W53OCc</a></p>&mdash; aoirint🎐 (@aoirint) <a href="https://twitter.com/aoirint/status/1373100834477666306?ref_src=twsrc%5Etfw">March 20, 2021</a></blockquote>
（このステッカーほしい）

スタートアップスクリプトは、新しい空っぽのVPSを契約した（単に契約済みの空っぽのVPSでも可）ときに初期設定をするためのシェルスクリプトです（初回起動時に自動実行してくれる）。環境変数みたいなものも注入することができます（スクリプトごとのプリセットに加えて、実行時に上書きもできる）。

本当は完全にSSHを使わなくていいように、自動アップデートやデータバックアップ、異常時のメール/Slack送信（おそらく異常時はバックアップからデータを復帰させてコンテナみたいにゼロベースで作り直すのがいい、と思うのだが、それならはじめからコンテナオーケストレーションサービスという選択肢があるのか..?）みたいなところまでスタートアップスクリプトに組み込めるとよいのですが、なかなかむずかしいので今回はそこまでやっていません。

さくらのVPSのスタートアップスクリプトには、公式が配布しているものに加えて、~~おそらくほとんど中の人なんじゃないかと思うが~~ユーザ間での共有機能があります。
ゲームサーバ（7 Days to Die、ARK、Minecraft、Factorio、Terrariaなど）やアプリケーションサーバ（Mastodon、GitLab Foss、Nextcloud、Mattermostなど）、SoftEther VPNなんかの有名どころはそろっているように思います。

余計なアプリケーションも立ちませんし、うまくすれば完全にGUIで操作できるので、ゲーム配信者（サーバエンジニア系でなくて、事務所にも所属してない）なんかにも有用なんじゃないかな（参加型配信とか、IP公開しなくてよくなるので）と思います（国内サーバだし）。
GUI化というのはVPSのコンソールだけでなくて、Basic認証 over HTTPSとか付けてゲームサーバ制御用のWebアプリを添付してもおもしろいかもしれません（固定のサブドメインはもらえるので）。

サーバを借りるときにつらいところはメモリが少ないこと、ストレージが少ないことだと思っています（ラックサーバ構築したことないのでそれとの比較は知らないけど）が、単アプリケーションの動作なら大丈夫な気がします。
ストレージが少ないといっても、画像・動画サーバみたいな使い方をしなければ大丈夫な容量はある気がします（最安で25GB、次点で50GB）。
それはAWS S3でやったほうがいいんでしょうね。

問題は年7000-円と費用が小さくはないことですかね。
配信を含む業務に関連した目的で、個人事業主なら経費にはなると思いますが（税金）。
競合にAWS Lightsailがありますが、費用はLightsailの方が安い気がします（料金体系がよくわかってないですが。サーバ起動自体は定額でも、通信量で追加課金とかされるんでしょうか？　それこそS3があるので、アプリケーションサーバの場合ちゃんと組んでいて、小-中規模ならそんなに通信しないとは思いますが）。
趣味用途の人は「お客様満足度調査」に回答するとQUOカードの抽選があったりするのでそのへんで回収を..。

- 料金体系：[https://vps.sakura.ad.jp/specification/](https://vps.sakura.ad.jp/specification/)

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">さくらお客様満足度調査、抽選当たった</p>&mdash; aoirint🎐 (@aoirint) <a href="https://twitter.com/aoirint/status/1315916620372406273?ref_src=twsrc%5Etfw">October 13, 2020</a></blockquote>

- [お客様満足度調査実施のお知らせ | さくらインターネット (2020/07)](https://www.sakura.ad.jp/information/announcements/2020/07/27/1968204228/)
