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

`vinanrra/7dtd-server`を使用します。

- 7 Days to Die α20.7
- パーティ人数は7 Days to Dieの仕様で8人まで
    - 9人以上の仲間で遊ぶ場合は複数パーティ組むなどの工夫が必要
    - もしくはDLL改変で上限を外せる可能性がある
      - [UnKnoWnCheaTs - Multiplayer Game Hacking and Cheats - View Single Post - [Question] 7 days to die - Max players in group/party](https://www.unknowncheats.me/forum/3398373-post4.html)
- RAM: 8人・マップPREGEN6kで8GB程度はほしい、1人でも最低4GB

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
