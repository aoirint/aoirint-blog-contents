---
title: 'サポーターズ ウインターハッカソン vol.7に参加しました'
date: '2021-03-19T11:29:04+09:00'
draft: false
channel: 技術ノート
category: 技術イベント
tags:
  - 技術イベント
  - ハッカソン
---
# サポーターズ ウインターハッカソン vol.7に参加しました

4人チーム（miniature-octo-guide）で、SpatialChatライクなUIでタブの音量調節をするChrome拡張を開発し、
約40人 17チームの中で最優秀賞をいただきました。

- [https://github.com/miniature-octo-guide/spatial-volume-controller](https://github.com/miniature-octo-guide/spatial-volume-controller)
- [https://talent.supporterz.jp/events/28d759c2-50b4-456d-889b-1f08abf6c053/](https://talent.supporterz.jp/events/28d759c2-50b4-456d-889b-1f08abf6c053/)

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">音声ボリュームをブラウザ上でコントロールすることで複数の講演を同時に視聴＆往来できる【Spatial Volume Control App】（電通大チーム）が最優秀賞を獲得！<br /><br />「講演を選び切れない」<br />「視聴しながら比較したい」<br /><br />そんな課題とニーズに応える素晴らしい作品・・・！<a href="https://twitter.com/hashtag/%E3%82%A6%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%BC%E3%83%8F%E3%83%83%E3%82%AB%E3%82%BD%E3%83%B3?src=hash&amp;ref_src=twsrc%5Etfw">#ウインターハッカソン</a><a href="https://twitter.com/hashtag/%E6%8A%80%E8%82%B2%E7%A5%AD?src=hash&amp;ref_src=twsrc%5Etfw">#技育祭</a> <a href="https://t.co/26H68ztPAi">https://t.co/26H68ztPAi</a> <a href="https://t.co/Le2fo8sAKC">pic.twitter.com/Le2fo8sAKC</a></p>&mdash; 楓博光@未来の技術者を育てる (@kaepon1219) <a href="https://twitter.com/kaepon1219/status/1365978116460478467?ref_src=twsrc%5Etfw">February 28, 2021</a></blockquote>

<blockquote class="twitter-tweet"><p lang="ja" dir="ltr">最優秀賞いただきました！<br />「Spatial Volume Control App」（Chrome拡張）<a href="https://t.co/6FgmhisC1d">https://t.co/6FgmhisC1d</a><a href="https://twitter.com/hashtag/%E6%8A%80%E8%82%B2%E7%A5%AD?src=hash&amp;ref_src=twsrc%5Etfw">#技育祭</a> <a href="https://twitter.com/hashtag/%E3%82%A6%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%BC%E3%83%8F%E3%83%83%E3%82%AB%E3%82%BD%E3%83%B3?src=hash&amp;ref_src=twsrc%5Etfw">#ウインターハッカソン</a> <a href="https://twitter.com/hashtag/%E3%82%A6%E3%82%A4%E3%83%B3%E3%82%BF%E3%83%BC%E3%83%8F%E3%83%83%E3%82%AB%E3%82%BD%E3%83%B3vol7?src=hash&amp;ref_src=twsrc%5Etfw">#ウインターハッカソンvol7</a> <a href="https://twitter.com/hashtag/%E3%82%B5%E3%83%9D%E3%83%BC%E3%82%BF%E3%83%BC%E3%82%BA?src=hash&amp;ref_src=twsrc%5Etfw">#サポーターズ</a></p>&mdash; aoirint🎐 (@aoirint) <a href="https://twitter.com/aoirint/status/1365964375568248833?ref_src=twsrc%5Etfw">February 28, 2021</a></blockquote>

開発に使うOSは3人がWindows、1人がLinuxでした（macOSのテスト環境も用意はあった）。
開発経験は、Web系（Pythonなど） 3人、Unity・マイコン系 1人でした。
ハッカソンは全員初参加でした。

開発アイデアを集めたのち、実装可能性を検討している中で、ベースをChrome拡張としました。
当初はOSのAPIを叩くネイティブアプリ（スタンドアロンソフトウェア）としての実装を検討していました。
Windows Core Audio、macOS CoreAudio.framework、Linux PulseAudioのAPIを叩くことを考えていましたが、
いくつかの理由でネイティブアプリでの実装を取りやめました。

- 参考資料が少ない
- Windowsでの動作検証で、Chromeタブごとの音量制御ができなさそうだった
  - プロセスごとの制御はできる
- マルチプラットフォームな実装が面倒
  - 各OSごとのAPI呼び出し
  - GUI実装

言語にはTypeScriptを選択しました。
TypeScriptの採用理由は、流行りの言語であること、型の記述によるデバッグ性・メンテナンス性が高いこと、あたりを考えていました。
全員簡単なJavaScriptを書いた経験はあったようですが、
テンプレートをいじった程度の簡単なコードを書いた経験があったわたしを除いてはTypeScriptの経験はなさそうでした。
しっかりしたChrome拡張を作った経験は全員ありませんでした。

準備期間で開発環境を用意しました（事前開発あり）。
Docker上で開発することを想定し、
GitHub上にOrganizationを作ってパブリックリポジトリでコードをホストし、
GitHub Actionsで自動コードチェック・ビルドする環境を整備しました。
実装課題はIssueを立て、
コードの共有は同一リモートリポジトリ内でブランチを切り、プルリクエストを作成し、
統合はプルリクエストのSquash Mergeですることにしました。
ドキュメントをNotion（1000ブロック制限付きのチーム）に作成し、
コミュニケーションはDiscord（通話あり）とハッカソン用Slackを使いました。

プロジェクトテンプレートの生成にはmazamachi/generator-chrome-extension-kickstart-typescriptを使いました。

- [https://github.com/mazamachi/generator-chrome-extension-kickstart-typescript](https://github.com/mazamachi/generator-chrome-extension-kickstart-typescript)

Linterにはts-standardを使いました。

- [https://github.com/standard/ts-standard](https://github.com/standard/ts-standard)

全員ブランチを切ってコミットし、pushするのには問題なく開発を始めることができました。
開発タスクはUIの実装、音量制御の実装、タブ画面表示の実装の3つに分割しました。
メンバーにタスクを割り振りましたが、あぶれたので自分はヘルパー的な立ち位置でいることにしました。
主な作業は実装方法の調査、開発相談/進捗管理、コードレビュー、コード統合でした。
TypeScript・モダンなECMAScriptには不慣れでしたが、プルリクエストのコードレビュー機能を活用して
書き換えてほしいコードを指摘するなどして開発を進めました。

反省点

- Notionは活用できなかった
  - READMEにもNotionにも開発手順を書いていたが分散しただけだったかもしれない
  - 時間の少ないハッカソンなので、チャット・通話ベースのコミュニケーションで十分だったかもしれない
  - 開発の打ち合わせみたいなコミュニケーションはもっと取っておきたかった
- ユニットテストほか自動テスト環境がなかった
  - Node.js/TypeScript環境に不慣れなため、コードテスト環境を事前に整備できなかった
- Issueは勝手に立たなかった・あまりGitHub上でコメントしてくれなかった
  - チャット・通話ベースでよかったのかもしれないが、開発過程を記録する意味で使ってほしかった
  - こういう形式での開発は自分もはじめてで、全員不慣れだったからというのもあると思われる
- マージ作業をメンバーに任せることができなかった
  - Linterの説明が不十分などのコミュニケーション不足、GitHub開発に不慣れ
  - 最終統合まで動いている様子が共有できず、メンバーに開発の先行きが見えなかった
- 拡張機能のフローが完成しなかった
  - アプリの体験までは作ることができたが、リソース解放処理などが未実装
- タブ画面表示が完成しなかった
  - 複雑なWebRTC関連の実装に手間取った
- Chrome Web Storeに公開できなかった
  - フローができていないので..

ハッカソン終了後は、対外的にコードを再利用しやすくするためライセンスを設定しました。
これまでのチーム開発ではコードをパブリックにする決定がなかなかできず、
コミュニケーションもとれない（とる機会がない）ので開発後にプライベートのまま放置してしまっていたのですが、
開発を始める前からパブリックにし、ライセンスを設定して
OSS化することでチームとしての意思決定を外れて勝手にコードを改修できる・あるいはプロジェクトをforkできるようになり、
この問題を解決できないかと考えています。
コミット履歴にハンドルネームを使うかどうか、などの面倒な問題もはじめからパブリックにすることで回避できます。
ライセンスの設定までは全員とコミュニケーションをとっておく必要があるので、できるなら開発を始める前に合意して、条文を含めておくのがいいように思います。
チームメンバー全員がプロジェクトに興味がある、反応できる状態がずっと続くとは限らないので、
IssueやPullRequestを立てたら反応してもらいたいところですが、なかなかむずかしいです
（GitHubのアカウント自体を乗り換えて捨ててしまうこともありますし）。

またハッカソン後、技育祭LTに向けても開発を進めました。
技育祭LT時点でタブ画面を表示する実装は完了しましたが、リソース解放処理などが未実装のため、Chrome Web Storeにはまだ公開していません。

また、SpatialChatがそうしているように、Webアプリとしての実装も可能なことにハッカソン中に気づきました。
SpatialChatは空間の共有がコンセプトですが、
このアプリはサーバが不要なスタンドアロン動作がコンセプトと考えているので、
微妙なところですが、Chrome拡張として強い権限を要求する状態にあるため、
導入しやすさの観点でWebアプリとして再実装することも検討しています。
