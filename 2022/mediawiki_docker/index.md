---
title: MediaWikiのセットアップ
date: '2022-02-03T07:00:00+09:00'
draft: false
channel: 技術ノート
category: Blogging
tags:
  - Blogging
  - Wiki
  - MediaWiki
---
# MediaWikiのセットアップ

## インストール

Docker Hub公式のDockerイメージが公開されている。

- [Mediawiki - Official Image | Docker Hub](https://hub.docker.com/_/mediawiki/)

### 事前準備

#### docker-compose.yml

```yaml
version: '3.8'
services:
  mediawiki:
    image: mediawiki:1.37
    restart: always
    ports:
      - '127.0.0.1:8000:80'
    links:
      - database
    volumes:
      - ./app/images:/var/www/html/images
      # After initial setup, download LocalSettings.php to the same directory as
      # this yaml and uncomment the following line and use compose to restart
      # the mediawiki service
      # - ./LocalSettings.php:/var/www/html/LocalSettings.php
      # - ./app/extensions:/var/www/html/extensions
  # This key also defines the name of the database host used during setup instead of the default "localhost"
  database:
    image: mariadb:10.7
    restart: always
    volumes:
      - database-data:/var/lib/mysql
    environment:
      # @see https://phabricator.wikimedia.org/source/mediawiki/browse/master/includes/DefaultSettings.php
      MYSQL_DATABASE: my_wiki
      MYSQL_USER: wikiuser
      MYSQL_PASSWORD: example
      MYSQL_RANDOM_ROOT_PASSWORD: 'yes'
volumes:
  database-data:
```

#### nginx設定ファイル (/etc/nginx/sites-enabled/wiki.example.com.conf）

```nginx
server {
    server_name wiki.example.com;

    proxy_set_header HOST $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # auth_basic "Authentication Required";
    # auth_basic_user_file /path/to/.htpasswd;

    location / {
        proxy_pass http://localhost:8000;
    }
}
```

### 手順

1. docker-compose.yml、nginx設定ファイルを作成
    - 必要に応じて`sudo certbot --nginx`でHTTPS対応
2. 添付ファイルのアップロード先ディレクトリを作成 `mkdir -p app/images`
3. `docker-compose up -d`でMediaWikiを起動
    - 起動時に`wikiexamplecom_mediawiki_1 is up-to-date`のようなログが表示されるので、`is`より前のMediaWikiコンテナ名を控える
4. ブラウザから初期設定
5. 設定ファイル`LocalSettings.php`をコピー
    - `sudo docker cp MediaWikiコンテナ名:/var/www/html/LocalSettings.php ./LocalSettings.php`
6. 拡張機能をコピー
    - `sudo docker cp MediaWikiコンテナ名:/var/www/html/extensions ./app/extensions`
7. docker-compose.ymlを開き、`./LocalSettings.php`、`./app/extensions`のコメントアウトを解除
8. コンテナを再作成
    - `sudo docker-compose up -d --force-recreate`

## 拡張機能のインストール

### インストール手順

1. 拡張機能の配布ファイル（tar.gz）をサーバにダウンロード
2. 配布ファイルを展開
3. 展開した拡張機能ディレクトリを`app/extensions`以下にコピー（`app/extensions/Math`のように）
    - Dockerコンテナ内では`/var/www/html/extensions/Math`のように配置される
4. `LocalSettings.php`に`wfLoadExtension( '拡張機能ディレクトリ名' );`のように追記
    - `wfLoadExtension( 'Math' );`
    - `wfLoadExtension( 'SyntaxHighlight_GeSHi' );`
5. 「メンテナンス：再起動」を実行
6. 「メンテナンス：データベース構造の更新」を実行

### 数式（Math）

- [Extension:Math - MediaWiki](https://www.mediawiki.org/wiki/Extension:Math)
- [Download MediaWiki extension - MediaWiki](https://www.mediawiki.org/wiki/Special:ExtensionDistributor/Math)

### シンタックスハイライト（SyntaxHighlight）

- [Extension:SyntaxHighlight - MediaWiki](https://www.mediawiki.org/wiki/Extension:SyntaxHighlight)
- [Download MediaWiki extension - MediaWiki](https://www.mediawiki.org/wiki/Special:ExtensionDistributor/SyntaxHighlight_GeSHi)

## 設定

### ロゴを変更

#### ロゴ変更 LocalSettings.php

```php
## The URL paths to the logo.  Make sure you change this from the default,
## or else you'll overwrite your logo when you upgrade!
#$wgLogos = [ '1x' => "$wgResourceBasePath/resources/assets/wiki.png" ];
$wgLogos = [ '1x' => "$wgResourceBasePath/images/logo.png" ];
```

#### ロゴ変更 手順

1. LocalSettings.phpを編集
2. ロゴ画像を`app/images/logo.png`に配置
3. 「メンテナンス：再起動」を実行

### 既定のタイムゾーンを変更

- [Manual:Timezone - MediaWiki](https://www.mediawiki.org/wiki/Manual:Timezone)

#### タイムゾーン変更 LocalSettings.php

```php
# Time zone
#$wgLocaltimezone = "UTC";
$wgLocaltimezone = "Asia/Tokyo";
```

#### タイムゾーン変更 手順

1. LocalSettings.phpを編集
2. 「メンテナンス：再起動」を実行

## メンテナンス

### 再起動

```bash
sudo docker-compose up -d --force-recreate
```

### データベース構造の更新

- [Manual:update.php - MediaWiki](https://www.mediawiki.org/wiki/Manual:Update.php)

```bash
sudo docker-compose exec -u 1000 mediawiki php maintenance/update.php
```
