---
title: SmokePing in Docker
date: '2021-08-22 09:45:00'
draft: false
category: Network
tags:
  - Network
  - SmokePing
---

# SmokePing in Docker

- https://hub.docker.com/r/dperson/smokeping

## docker-compose.yml
```yaml
version: '3.9'
services:
  smokeping:
    image: dperson/smokeping:latest
    restart: always
    ports:
      - '{SMOKEPING_PORT:-127.0.0.1:8000}:80'
    volumes:
      - './config:/etc/smokeping:ro'
      - './data:/var/lib/smokeping'
    environment:
      TZ: Asia/Tokyo
```

## config/config.d/Targets
```smokeping
*** Targets ***

probe = FPing

menu = Top
title = Network Latency Grapher
remark = Welcome to the SmokePing website of xxx Company. \
         Here you will learn all about the latency of our network.

+ Home

menu = Home
title = Home Network
#parents = owner:/Test/James location:/

++ Router

menu = Router (192.168.0.1)
title = Router (192.168.0.1)
host = 192.168.0.1

++ MyServer

menu = MyServer (192.168.0.2)
title = MyServer (192.168.0.2)
host = 192.168.0.2
#alerts = someloss
```
