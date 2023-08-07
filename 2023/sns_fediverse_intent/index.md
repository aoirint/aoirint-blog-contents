---
title: 'SNS・Fediverseの投稿インテントURL'
date: '2023-08-07T15:50:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: SNS
tags:
  - SNS
  - Fediverse
  - Misskey
  - Mastodon
  - Twitter
---
# SNS・Fediverseの投稿インテントURL

Twitter (X)、Misskey、Mastodonには、URLのGETパラメータに投稿本文などを付けて、ミニブログの入力を補助する機能（投稿インテントURL機能）があります。

この記事では、各サービス・ソフトウェアの投稿インテントURLの仕様について記載します。

## Twitter Web Intent (2023-07-19時点)

- ドキュメント: [Web Intent | Docs | Twitter Developer Platform](https://developer.twitter.com/en/docs/twitter-for-websites/tweet-button/guides/web-intent)

```
https://twitter.com/intent/tweet
```

|GETパラメータ|備考|
|:--|:--|
|text||
|url||
|hashtags||
|via||
|related||
|in_reply_to||

## Misskey 共有フォーム (v13.13.2時点, 2023-07-13)

- ドキュメント: [共有フォーム | Misskey Hub](https://misskey-hub.net/docs/features/share-form.html)

[Misskey.io](https://misskey.io/)を例とします。

```
https://misskey.io/share?text=hello
```

|GETパラメータ|備考|
|:--|:--|
|title||
|text||
|url||
|replyId||
|replyUri||
|renoteId||
|renoteUri||
|visibility||
|localOnly||
|visibleUserIds||
|visibleAccts||
|fileIds||

## Mastodon (v4.1.4時点, 2023-07-08)

- ドキュメント: 見つけられなかった
  - 実装はここ: [https://github.com/mastodon/mastodon/blob/3f5af768c8f1401f77d14ad5b6aeccdb7e02a9f0/app/helpers/application_helper.rb#L196-L204](https://github.com/mastodon/mastodon/blob/3f5af768c8f1401f77d14ad5b6aeccdb7e02a9f0/app/helpers/application_helper.rb#L196-L204)

[mstdn.aoirint.com](https://mstdn.aoirint.com/)を例とします。

```
https://mstdn.aoirint.com/share?text=hello
```

|GETパラメータ|備考|
|:--|:--|
|title|title, text, urlの順で半角スペース結合されたものがトゥート本文としてデフォルト入力される|
|text||
|url||
|visibility|public, unlisted, private, directのいずれか|

## 課題と関連サービス

Fediverseでは、従来のTwitter (X)やFacebookと異なり、
ユーザが大規模サーバからお一人様サーバまで様々なサーバに属するため、
ユーザによってインテントURLが変わるという課題があります。
また、ある程度のデファクトスタンダードはあるものの、
ActivityPubに対応したマイクロブログ実装の間で統一された仕様がないという課題もあります。

前者の課題に関連したサービスには、以下のようなものがあります（いずれもMastodon向け）。

### Mastoshare

- [Mastoshare](https://mastoshare.net/)
- [GitHub naaaaaaaaaaaf/mastoshare](https://github.com/naaaaaaaaaaaf/mastoshare)

ブラウザのlocalStorageにサーバ一覧を保存する形式。

```
https://mastoshare.net/share?text=Hello%20Mastoshare!&url=https://mastoshare.net
```

### マストポータル

- [マストポータル](https://mastportal.info/share)

サービス管理者がサーバ一覧を管理する形式。

## 関連URL

- [URLパラメータを用いて Mastodon に投稿内容を渡して投稿フォームを表示する方法 - 約束の地](https://obel.hatenablog.jp/entry/20230217/1676624400)
- [Intent/share URLs · Issue #442 · mastodon/mastodon](https://github.com/mastodon/mastodon/issues/442)
- [URL scheme for remote follow, share buttons · Issue #2291 · mastodon/mastodon](https://github.com/mastodon/mastodon/issues/2291)
- [Use web-based protocol handlers to make following across instances easier · Issue #19679 · mastodon/mastodon](https://github.com/mastodon/mastodon/issues/19679)
- [Mastodon向け簡易シェアボタン - Qiita](https://qiita.com/mod_poppo/items/d80ff225b4cc93318ee8)
- [https://mstdn.aoirint.com/@aoirint/110740279312480478](https://mstdn.aoirint.com/@aoirint/110740279312480478)
