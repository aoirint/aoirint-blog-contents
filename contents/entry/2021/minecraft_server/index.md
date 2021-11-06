---
title: Minecraft Server
date: '2021-11-06 17:50:00'
draft: false
category: Game
tags:
  - Game
  - Minecraft
  - 'Minecraft Server'
---
# Minecraft Server

- <https://github.com/itzg/docker-minecraft-server>
- <https://feedback.minecraft.net/hc/en-us/sections/360001186971-Release-Changelogs>

## docker-compose.yml

```yaml
version: "3.9"

services:
  minecraft:
    image: itzg/minecraft-server
    ports:
      - "${SERVER_PORT}:25565"
    environment:
      EULA: "TRUE"
      OVERRIDE_SERVER_PROPERTIES: 'false' # overwrite on every container start if true
      TZ: ${TZ}
      TYPE: ${TYPE} # https://github.com/itzg/docker-minecraft-server#server-types
      VERSION: ${VERSION} # https://feedback.minecraft.net/hc/en-us/sections/360001186971-Release-Changelogs
      SERVER_NAME: ${SERVER_NAME}
      MOTD: ${MOTD}
      ENABLE_WHITELIST: ${ENABLE_WHITELIST}
      WHITELIST: ${WHITELIST}
      OPS: ${OPS}
      SPAWN_PROTECTION: ${SPAWN_PROTECTION}
      VIEW_DISTANCE: ${VIEW_DISTANCE}
      SEED: ${SEED}
      DIFFICULTY: ${DIFFICULTY}
      MODE: ${MODE}
      PVP: ${PVP}
      LEVEL_TYPE: ${LEVEL_TYPE}
      GENERATOR_SETTINGS: ${GENERATOR_SETTINGS}
      ONLINE_MODE: ${ONLINE_MODE}
    tty: true
    stdin_open: true
    restart: unless-stopped
    volumes:
      - ./data:/data
```

## .env

```env
SERVER_PORT=0.0.0.0:25565
TZ=Asia/Tokyo
TYPE=VANILLA
VERSION=1.17.1
SERVER_NAME=MyServer
MOTD=A Vanilla Minecraft Server powered by Docker
ENABLE_WHITELIST=true
WHITELIST=user1,user2
OPS=user1
SPAWN_PROTECTION=0
VIEW_DISTANCE=
SEED=
DIFFICULTY=normal
MODE=survival
PVP=true
LEVEL_TYPE=
GENERATOR_SETTINGS=
ONLINE_MODE=true
```

### スーパーフラット

- <https://github.com/itzg/docker-minecraft-server#level-type-and-generator-settings>
- <https://minecraft.fandom.com/wiki/Superflat>

** ※ 1.17.1でうまく動きませんでした（常にデフォルトのフラットワールドが生成される）。
代わりに、クライアントでワールドを生成してからサーバにコピーする方法が使えます。 **

- <https://bugs.mojang.com/browse/MC-195468>

```env
LEVEL_TYPE=flat

# Overworld without structure
# for <1.13
# GENERATOR_SETTINGS=minecraft:bedrock,59*minecraft:stone,3*minecraft:dirt,minecraft:grass_block;minecraft:plains

# for 1.13+
GENERATOR_SETTINGS={"biome":"minecraft:plains","lakes":false,"features":false,"layers":[{"block":"minecraft:bedrock","height":1},{"block":"minecraft:stone","height":59},{"block":"minecraft:dirt","height":3},{"block":"minecraft:grass_block","height":1}],"structures":{"structures":{}}}
```

- JSON生成: <https://misode.github.io/world/>
    - <https://github.com/itzg/docker-minecraft-server/issues/999#issuecomment-907849644>

```shell
# https://misode.github.io/world/ で生成したjsonをsuperflat.jsonとして保存

jq -c '.dimensions."minecraft:overworld".generator.settings' superflat.json
```

## その他

統合版（Bedrock Edition）からJava版サーバに接続するための非公式プロキシGeyserが公開されている。

- [GeyserMC](https://github.com/GeyserMC/Geyser)
- [GeyserConnect](https://github.com/GeyserMC/GeyserConnect)
