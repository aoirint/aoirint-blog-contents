---
title: '7 Days to Die Dedicated Server in Docker'
date: '2023-04-24T21:00:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Game
tags:
  - Game
  - '7 Days to Die'
---
# 7 Days to Die Dedicated Server in Docker

- [vinanrra/7dtd-server - Docker Image | Docker Hub](https://hub.docker.com/r/vinanrra/7dtd-server)
- [vinanrra/Docker-7DaysToDie: 7 days to die server using LinuxGSM in Docker with backups, monitor, auto-installable mods and more](https://github.com/vinanrra/Docker-7DaysToDie)
- [aoirint/seven-days-server-test: 7 Days to Die Dedicated ServerのDocker Compose構成テスト](https://github.com/aoirint/seven-days-server-test)

`vinanrra/7dtd-server`を使用します。

- 7 Days to Die α20.7
- パーティ人数は7 Days to Dieの仕様で8人まで
    - 9人以上の仲間で遊ぶ場合は複数パーティ組むなどの工夫が必要
    - もしくはDLL改変で上限を外せる可能性がある
      - [UnKnoWnCheaTs - Multiplayer Game Hacking and Cheats - View Single Post - [Question] 7 days to die - Max players in group/party](https://www.unknowncheats.me/forum/3398373-post4.html)
- RAM: 8人・マップPREGEN6kで8GB程度はほしい、1人でも最低4GB

## docker-compose.yml

```yaml
# Based on https://github.com/vinanrra/Docker-7DaysToDie/blob/d5dbbb4e2ec65614b985d653f51725932d171dc7/docs/usage.md
version: '3.8'
services:
  7dtdserver:
    image: "vinanrra/7dtd-server:v0.5.0"
    restart: always # INFO - NEVER USE WITH START_MODE=4 or START_MODE=0
    environment:
      - START_MODE=1 # Change between START MODES
      - VERSION=alpha20.7 # Change between 7 days to die versions
      - PUID=1000 # Remember to use same as your user
      - PGID=1000 # Remember to use same as your user
      - TimeZone=Asia/Tokyo # Optional - Change Timezone
      - BACKUP=NO # Optional - Backup server at 5 AM
      - MONITOR=NO # Optional - Keeps server up if crash
    volumes:
      - ./volumes/7DaysToDie:/home/sdtdserver/.local/share/7DaysToDie/
      - ./volumes/LGSM-Config:/home/sdtdserver/lgsm/config-lgsm/sdtdserver/
      - ./volumes/ServerFiles:/home/sdtdserver/serverfiles/ # Optional - serverfiles folder
      - ./volumes/log:/home/sdtdserver/log/ # Optional - Logs folder
      - ./volumes/backups:/home/sdtdserver/lgsm/backup/ # Optional - If BACKUP=NO, backups folder
    ports:
      - 26900:26900/tcp # Default game ports
      - 26900:26900/udp # Default game ports
      - 26901:26901/udp # Default game ports
      - 26902:26902/udp # Default game ports
      # - 8080:8080/tcp # OPTIONAL - WEBADMIN
      # - 8081:8081/tcp # OPTIONAL - TELNET
      # - 8082:8082/tcp # OPTIONAL - WEBSERVER https://7dtd.illy.bz/wiki/Server%20fixes
```

## 7 Days to Dieサーバ本体のアップデート

- [https://github.com/vinanrra/Docker-7DaysToDie/blob/9327daecb66d412b520532f739111955e7985aa5/docs/parameters.md#start-modes](https://github.com/vinanrra/Docker-7DaysToDie/blob/9327daecb66d412b520532f739111955e7985aa5/docs/parameters.md#start-modes)

`docker-compose.yml`の`environment`を以下のように変更します。

- `VERSION`: 目的のバージョン
- `START_MODE`: `3`

`docker compose up -d --force-recreate`を実行し、アップデート完了まで待機します。

`docker compose logs -f`でログを確認します。

アップデートが完了し、サーバが起動したら、`START_MODE`を`1`に戻し、サーバを再起動します。

## サーバ設定

サーバ設定を変更するには、`volumes/ServerFiles/sdtdserver.xml`を編集します。

### セキュリティ

#### ServerPassword

> Password to gain entry to the server

サーバに接続するときに、パスワードを要求します。

#### ServerVisibility

> Visibility of this server: 2 = public, 1 = only shown to friends, 0 = not listed. As you are never friend of a dedicated server setting this to "1" will only work when the first player connects manually by IP.

- `0`
- `1`
- `2`

#### EACEnabled

> Enables/Disables EasyAntiCheat

- `true`
- `false`

#### BuildCreate

> cheat mode on/off

- `true`
- `false`

#### PersistentPlayerProfiles

> If disabled a player can join with any selected profile. If true they will join with the last profile they joined with

- `true`
- `false`

### 負荷

#### ServerMaxPlayerCount

> Maximum Concurrent Players

#### ServerMaxWorldTransferSpeedKiBs

> Maximum (!) speed in kiB/s the world is transferred at to a client on first connect if it does not have the world yet. Maximum is about 1300 kiB/s, even if you set a higher value.

#### MaxSpawnedZombies

> This setting covers the entire map. There can only be this many zombies on the entire map at one time. Changing this setting has a huge impact on performance.

#### MaxSpawnedAnimals

> If your server has a large number of players you can increase this limit to add more wildlife. Animals don't consume as much CPU as zombies. NOTE: That this doesn't cause more animals to spawn arbitrarily: The biome spawning system only spawns a certain number of animals in a given area, but if you have lots of players that are all spread out then you may be hitting the limit and can increase it.

#### ServerMaxAllowedViewDistance

> Max viewdistance a client may request (6 - 12). High impact on memory usage and performance.

### 難易度

#### GameWorld

> "RWG" (see WorldGenSeed and WorldGenSize options below) or any already existing world name in the Worlds folder (currently shipping with e.g. "Navezgane", "PREGEN01", ...)

- `RWG`: ランダム生成
- `Navezgane`
- `PREGEN6k`: 固定マップ

#### GameDifficulty

> 0 - 5, 0=easiest, 5=hardest

#### DropOnDeath

> 0 = nothing, 1 = everything, 2 = toolbelt only, 3 = backpack only, 4 = delete all

#### AirDropMarker

> Sets if a marker is added to map/compass for air drops.

- `true`
- `false`

#### PlayerKillingMode

> Player Killing Settings (0 = No Killing, 1 = Kill Allies Only, 2 = Kill Strangers Only, 3 = Kill Everyone)

- `0`
- `1`
- `2`
- `3`

## ゲーム内コマンド
