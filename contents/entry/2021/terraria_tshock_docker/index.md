---
title: Terraria TShockサーバをdocker-composeで立てる
date: '2021-08-22 05:25:00'
draft: false
category: Game
tags:
  - Game
  - Terraria
---

# Terraria TShockサーバをdocker-composeで立てる

- <https://github.com/Pryaxis/TShock>
- <https://hub.docker.com/r/ryshe/terraria/>

上記URLから最新（もしくは対応する）バージョンを確認して、イメージタグのバージョンを変更する。

既存のワールドを使用する場合は、`./data/worlds/`に`.wld`ファイルを配置し、`WORLD_FILENAME`を変更する。

## docker-compose.yml
```yaml
version: '3.9'
services:
  terraria:
      image: ryshe/terraria:tshock-1.4.2.3-4.5.5
      tty: true
      stdin_open: true
      restart: always
      ports:
        - '7777:7777'
      volumes:
        - '${WORLD_DIR:-./data/worlds}:/root/.local/share/Terraria/Worlds'
        - '${LOG_DIR:-./data/logs}:/tshock/logs'
        - '${BACKUP_DIR:-./data/backups}:/tshock/backups'                       
        - '${PLUGIN_DIR:-./data/plugins}:/plugins'
      environment:
        TZ: 'Asia/Tokyo'
        WORLD_FILENAME: 'MyWorldName.wld'
```

## ワールドの新規作成
autocreateオプションにはワールドサイズを数値で指定する（1: Small, 2: Medium, 3: Large）。
ここではSmallを指定している。

環境変数WORLD_FILENAMEが設定されていると実行に失敗するので、`-e WORLD_FILENAME=`で外しておく。

```shell
docker-compose run --rm -e WORLD_FILENAME= terraria -world /root/.local/share/Terraria/Worlds/MyWorldName.wld -autocreate 1
```

作成完了後、サーバが起動し、サーバコンソールが開く。

バックグラウンド起動にするため、ここでは一旦サーバを停止しておく。

## サーバの起動
```shell
docker-compose up -d
docker-compose logs -f

tail -f data/logs/*.log
```

## サーバコンソールを開く
Dockerコンテナ名を調べる。ここでは、`tshock_terraria_1`とする。

```shell
docker-compose ps
```

以下コマンドでサーバコンソールが開く。
Ctrl+cするとサーバが停止してしまうので、脱出するときはCtrl+p Ctrl+qを押す。

```shell
docker attach tshock_terraria_1
```

## 設定・権限の変更
TShockにはバニラと異なる細かいデフォルト設定や権限機能があるので注意。
設定では、デフォルトで初期スポーン地点保護が有効化、墓生成が無効化されている。
権限では、デフォルトでNPCの部屋割り当て、ボス召喚などが無効化されている。
また、TShockのバージョンによってコマンドや設定方法に差異がある。

### ./data/worlds/config.json

#### ServerPassword
サーバアクセス時に入力を要求するパスワード

#### SpawnProtection
初期スポーン保護

#### DisableTombstones
死亡時の墓生成の無効化


### 権限の変更
TShockサーバにはゲームキャラクタと独立したアカウントの概念があり、ユーザの権限はアカウントが属するグループに基づく。

サーバ初回起動時に生成される`./data/worlds/setup-code.txt`内のパスワードを使い、`/setup <setup-code>`を実行することで、一時的に最高権限のsuperadminになる。

ここで自分のアカウントにsuperadmin権限を付与すれば、`/login`で自分のアカウントにログインしたときに最高権限になる。Setup Codeは一度しか使えないため注意。

superadmin不在でsetup codeを失った場合、サーバコンソールを使うか、TShockのDBを直接編集してsuperadmin権限を付与する。

#### 全ユーザへの主要な制限の解除

大規模サーバでは負荷や荒らし等が問題になるかもしれないので、身内サーバ向けの設定。

guestグループは非ログインユーザを含むユーザ、defaultグループはログイン済みユーザ。

```terraria
group addperm guest tshock.ignore.*
group addperm guest tshock.tp.rod
group addperm guest tshock.world.editspawn
group addperm guest tshock.npc.summonboss
group addperm guest tshock.world.movenpc
group addperm guest tshock.npc.hurttown
```

- <https://www.reddit.com/r/Terraria/comments/354yle/tshock_help_turning_off_all_restrictions/>
- <https://tshock.readme.io/docs/permissions>
- <https://ch.nicovideo.jp/owenhousou/blomaga/ar580919>
  - <https://web.archive.org/web/20210821201236/https://ch.nicovideo.jp/owenhousou/blomaga/ar580919> （ニコニコユーザブロマガがサービス終了するのでアーカイブ）
