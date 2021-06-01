---
canonical_url: ./
title: Wiki.jsのセットアップ
date: '2021-06-01 05:00:00'
draft: false
category: Blogging
tags:
  - Blogging
  - Wiki
  - Wiki.js
---

# Wiki.jsのセットアップ

## メモリ消費量

```shell
$ docker stats
CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
e225b6b77209   wikijs_wiki_1   0.01%     127.6MiB / 7.681GiB   1.62%     13.1MB / 22.7MB   44.5MB / 1.45MB   11
13d085313406   wikijs_db_1     0.00%     44.57MiB / 7.681GiB   0.57%     6.64MB / 4.09MB   4.27MB / 102MB    8
```

初期状態で150-200MiB程度の消費量と思われる。

下記の環境をサーバとして、別端末からNATループバックによる接続を試したところ、ページ遷移時のロードや記事の保存に少しだけ時間がかかるように思われたが、Wikiサイト自体がなかなか開かない、というような重さではなかった。

```plain
CPU: Intel(R) Core(TM) i5-4570 CPU @ 3.20GHz
Memory: 8GB
Storage: SSD
OS: Ubuntu Desktop 18.04
```

## docker-compose.yaml

```yaml
version: "3.9"
services:
  db:
    image: postgres:13-alpine
    environment:
      POSTGRES_DB: wiki
      POSTGRES_PASSWORD: wikijsrocks
      POSTGRES_USER: wikijs
    restart: unless-stopped
    volumes:
      - db-data:/var/lib/postgresql/data

  wiki:
    image: requarks/wiki:2
    depends_on:
      - db
    environment:
      DB_TYPE: postgres
      DB_HOST: db
      DB_PORT: 5432
      DB_USER: wikijs
      DB_PASS: wikijsrocks
      DB_NAME: wiki
    restart: unless-stopped
    ports:
      - "127.0.0.1:8000:3000"

volumes:
  db-data:
```

- https://hub.docker.com/r/requarks/wiki
- https://hub.docker.com/_/postgres

基本的に初期設定で問題ない。

wikiサービスのポートの割り当てはホストのnginxからリバースプロキシすることを想定している。


## 初期設定
ポートにアクセスすると初期（管理者）アカウントの設定を求められる。

![wikijs_configure_first_account.png](/wikijs/wikijs_configure_first_account.png)

管理者アカウント作成後、インデックスページの作成を求められる。

![wikijs_first_page_1.png](/wikijs/wikijs_first_page_1.png)
![wikijs_first_page_2.png](/wikijs/wikijs_first_page_2.png)
![wikijs_first_page_3.png](/wikijs/wikijs_first_page_3.png)

## ロケール
Wikiページの言語にあたる、ロケールの設定が悩ましいところ。

言語を変えるとUIの言語だけでなくページ作成時のデフォルトURLプレフィクス（`/en/PAGE`とか`/ja/PAGE`）が変わる。

マニュアル用のスクリーンショットは`en`で撮りたいが、ページの言語は`ja`なのでプレフィクスは`/ja`にしたい、というのは都度切り替えないとできないかもしれない。

![wikijs_locale_en.png](/wikijs/wikijs_locale_en.png)

![wikijs_locale_ja.png](/wikijs/wikijs_locale_ja.png)


## 非公開Wiki
- https://docs.requarks.io/groups (Guides > Private Wiki)

管理画面の`Users > Groups > Guests`のPermissionsをすべて無効にする。

![wikijs_users_groups.png](/wikijs/wikijs_users_groups.png)

![wikijs_group_permissions.png](/wikijs/wikijs_group_permissions.png)


権限のないページを開くとUnauthorizedページが表示される。

![wikijs_unauthorized.png](/wikijs/wikijs_unauthorized.png)


## Gitリポジトリによる管理

管理画面の`Modules > Storage > Git`から、WikiデータをリモートGitリポジトリにバックアップするように設定できる。
またバックアップだけでなく、Gitリポジトリ側の変更をpullすることもでき、双方向に変更をやり取りすることもできるらしい。

パフォーマンスの問題で、同期のタイミングはページ更新直後ではなく、タイマーで行われるので注意。

また、Bi-directionalの場合リモート側にあらかじめブランチを作成しておく必要がある。
これは空コミットでなくてもよい。
ブランチが存在しない場合、以下のエラーが発生する。

```
Invalid branch! Make sure it exists on the remote first.
```

Gitストレージを設定する前に投稿した記事や画像は変更が加えられるまで同期されない。
条件は不明だが、内部エラーの影響か一部の画像ファイルがバージョン管理されないことがあった。
再アップロードで解消するが若干の注意を要するかもしれない。

![wikijs_git_storage.png](/wikijs/wikijs_git_storage.png)


Markdownページは、以下のようなYAMLヘッダ（Front Matter）がついたファイルとして保存される。

```plain
---
title: Wiki.jsのセットアップ
description: 
published: true
date: 2021-06-01T19:27:44.133Z
tags: wikijs, wiki
editor: markdown
dateCreated: 2021-06-01T18:48:16.731Z
---

# Wiki.jsのセットアップ

```

## GitHub Pages

そのままGitHub上でGitHub Pagesを有効にすれば、
デフォルト設定のJekyllで生成されたHTMLがホストされる。


## リファラを送らない

`Site > Theme > Code Injection > Head HTML Injection`に設定する。

```html
<!-- Do not send referrer -->
<meta name="referrer" content="no-referrer">
```
