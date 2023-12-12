---
title: 'メモアプリについて'
date: '2021-03-19T09:47:12+09:00'
draft: false
channel: ダイアリー
category: 怪文書
tags:
  - 怪文書
  - メモ
---
# メモアプリについて

メモ書きにはSimplenoteとJoplin、Atomを主に使っています。

WYSIWYGの選択肢もありますが、基本的にはMarkdownが好きです。
Sphinxを使う関係でreStructuredTextを書いたりもしましたが、Markdownの方がずっと書いているので慣れています。
しかしMarkdownは方言が多く、込み入った機能を使おうとすると安定しないのが難点です。
ここに記事を書くにあたって脚注を使ってみようとしたのですが、
Simplenoteでは表示できず、
AtomのMarkdown Preview Plusでは手動で機能を有効化する必要があるようです。
Joplinもデフォルト有効のオプションみたいでしたが、ツールによって安定しないのは使いづらいです。

- [https://atom.io/packages/markdown-preview-plus](https://atom.io/packages/markdown-preview-plus)

SimplenoteはAndroid、iOS、Linux、MacOS、Windowsすべてから、ブラウザと認証情報さえあればすぐにアクセスできて、モバイルアプリもあるのが強みなように思います。
また、テキストオンリーということで動作の高速化を図っているらしいです。
ただし、Simplenoteに大量のメモを保存したことはない（200件程度）ので実際に大量に保存した場合（1000件以上）の動作は不明です。
Simplenoteは、必要なときにメモをとって、必要なくなったら削除する or 念のために残しておく、くらいの使い方が向いている気がします。

- [https://app.simplenote.com/](https://app.simplenote.com/)

Joplinは、EvernoteライクなOSSです。
メモファイルはDropbox、OneDrive、S3ほかを使って同期できます。
テキストオンリーならばそれほど容量を消費しないという仮定に立てば、Dropboxライトユーザーなわたしには同期先としてDropboxがいい気がしています。
同期時は差分データをすべてダウンロードしてくるため、同期に時間がかかるのが難点です。
また、プロキシ非対応なのが不便です。
透過プロキシを使うことで強制的にHTTPプロキシを通過させることはできますが、便利とはいいがたいです。

- [https://github.com/laurent22/joplin/issues/164](https://github.com/laurent22/joplin/issues/164)
- [https://github.com/wadahiro/go-transproxy](https://github.com/wadahiro/go-transproxy)

Atomはメインのテキストエディタですが、Electronベースなので若干起動が遅いです。
コードやドキュメントを書くのに使い慣れているので、メモ書きにも使います。
ファイルの保存先選定に困るので、たいてい保存するときは他のアプリに移動しますが..

Notionも少し使いましたが、前記事の理由で使い続けるのはむずかしいです。
Confluenceは使い方がわかっていません（個人で使うには機能が多すぎるかも）。
Growiはそれなりによさそうなのですが、非公開利用には自前でホストする必要があったり、RAM消費が大きかったり、UI的に使い込めていない部分があります。
esa.ioは気になっていますが、Crowi/Growiベースに見えるので日常メモに向いているか不安があります。
