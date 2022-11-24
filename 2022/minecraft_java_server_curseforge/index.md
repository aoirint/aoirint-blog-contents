---
title: 'CurseForge Modpackを導入したMinecraft Java版 サーバ in Docker'
date: '2022-10-19T01:15:00+09:00'
updated: '2022-10-19T01:15:00+09:00'
draft: false
channel: 技術ノート
category: Game
tags:
  - Game
  - Minecraft
  - 'Minecraft Server'
---
# CurseForge Modpackを導入したMinecraft Java版 サーバ in Docker

- 前回の記事: [https://blog.aoirint.com/entry/2021/minecraft_server/](https://blog.aoirint.com/entry/2021/minecraft_server/)
- [https://hub.docker.com/r/itzg/minecraft-server](https://hub.docker.com/r/itzg/minecraft-server)
- [https://github.com/itzg/docker-minecraft-server](https://github.com/itzg/docker-minecraft-server)
- 例: [https://github.com/aoirint/minecraft-server-pixelmon-cj](https://github.com/aoirint/minecraft-server-pixelmon-cj)


## 2021年12月に確認されたLog4j 脆弱性（Log4jShell）への対応（Minecraft 1.7～1.18を使用する場合）

Minecraft公式から、旧バージョンにおける脆弱性対応手順が案内されています。

- [https://www.minecraft.net/ja-jp/article/important-message--security-vulnerability-java-edition-jp](https://www.minecraft.net/ja-jp/article/important-message--security-vulnerability-java-edition-jp)

### Minecraftサーバー

この記事で使用するDockerイメージ`itzg/minecraft-server`では、自動的に脆弱性への対応が適用されます。

- [https://github.com/itzg/docker-minecraft-server#mitigated-log4jshell-vulnerability](https://github.com/itzg/docker-minecraft-server#mitigated-log4jshell-vulnerability)

### Minecraftクライアント（CurseForge Appに管理された環境）

CurseForge Appでは、Minecraft向けに対応するパッチが適用されているようです（CurseForge Appが管理するMinecraft公式ランチャーのバージョンを脆弱性対応した新しいものに設定している？）。

> We already released a patch to the app

- [https://support.overwolf.com/en/support/solutions/articles/9000196615-curseforge-known-issues#Minecraft](https://support.overwolf.com/en/support/solutions/articles/9000196615-curseforge-known-issues#Minecraft)

### Mod

CurseForge上で脆弱性の影響を受けるModは削除されているようです（独自にlog4jを同梱していたり、特殊な実装をしていたりするものしか影響を受けない気がする）。

> to our best knowledge, any vulnerable mod was removed

- [https://support.overwolf.com/en/support/solutions/articles/9000196615-curseforge-known-issues#Minecraft](https://support.overwolf.com/en/support/solutions/articles/9000196615-curseforge-known-issues#Minecraft)


## Modpackを導入して新規ワールド生成

CurseForge Appまたは`curseforge.com`から、導入するModpackのServer Pack `serverpack000_my_mod_pack.zip` をダウンロードしてください。

以下では例としてMinecraft 1.12.2の環境ですが、Modpackに対応した本体バージョンに変更してください。

- Minecraft公式リリースノート: [https://feedback.minecraft.net/hc/en-us/sections/360001186971-Release-Changelogs](https://feedback.minecraft.net/hc/en-us/sections/360001186971-Release-Changelogs)

### docker-compose.yml

```yaml
version: "3.9"

services:
  minecraft:
    image: itzg/minecraft-server:2022.11.0-java8-jdk
    ports:
      - "${HOST_MINECRAFT_PORT}:25565"
      # - "${HOST_DYNMAP_PORT}:8123"
    environment:
      EULA: "TRUE"
      TZ: ${TZ}
      TYPE: "CURSEFORGE" # https://github.com/itzg/docker-minecraft-server#server-types
      CF_SERVER_MOD: /modpacks/serverpack000_my_mod_pack.zip
      VERSION: "1.12.2" # https://feedback.minecraft.net/hc/en-us/sections/360001186971-Release-Changelogs
      DIFFICULTY: ${DIFFICULTY}
      SERVER_NAME: ${SERVER_NAME}
      ENABLE_WHITELIST: ${ENABLE_WHITELIST}
      WHITELIST: ${WHITELIST}
      OPS: ${OPS}
      SPAWN_PROTECTION: ${SPAWN_PROTECTION}
      VIEW_DISTANCE: ${VIEW_DISTANCE}
      SEED: ${SEED}
      MODE: ${MODE}
      PVP: ${PVP}
      LEVEL_TYPE: ${LEVEL_TYPE}
      GENERATOR_SETTINGS: ${GENERATOR_SETTINGS}
      ONLINE_MODE: ${ONLINE_MODE}
    tty: true
    stdin_open: true
    restart: always
    volumes:
      - ./data:/data
      - ./modpacks:/modpacks
```

### .env

```env
HOST_MINECRAFT_PORT=0.0.0.0:25565
HOST_DYNMAP_PORT=127.0.0.1:8123
TZ=Asia/Tokyo
SERVER_NAME=my-mod-pack-server
MOTD=
ENABLE_WHITELIST=false
WHITELIST=
OPS=
SPAWN_PROTECTION=0
VIEW_DISTANCE=
SEED=
DIFFICULTY=hard
MODE=survival
PVP=true
LEVEL_TYPE=normal
GENERATOR_SETTINGS=
ONLINE_MODE=true
```

- HOST_MINECRAFT_PORT: 外部に直接公開（TCP）
- HOST_DYNMAP_PORT: リバースプロキシでHTTPS化を想定してローカルループバックアドレスにバインド
- WHITELIST, OPS: カンマ区切りでプロフィール名を記述


## サーバ側Modの追加

ModPackへの追加は、初期化以外のタイミングでは反映されないため、`./data`ディレクトリの`mods`フォルダに、通常のMod導入手順と同様に追加します。

上の`.env`では、ブラウザマップModのDynmapForgeをの追加導入を想定しています。

- [https://github.com/webbukkit/dynmap](https://github.com/webbukkit/dynmap)
- [https://www.curseforge.com/minecraft/mc-mods/dynmapforge](https://www.curseforge.com/minecraft/mc-mods/dynmapforge)
