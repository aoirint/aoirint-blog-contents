---
title: 'WordPress'
date: '2021-11-13T13:00:00+09:00'
draft: false
channel: 技術ノート
category: Blogging
tags:
  - WordPress
  - Blogging
---
# WordPress

- [https://hub.docker.com/_/wordpress](https://hub.docker.com/_/wordpress)

## docker-compose.yml

```yaml
version: "3.9"

services:
  wordpress:
    image: wordpress:5.8.2-apache
    restart: always
    ports:
      - "${SERVER_PORT}:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress:/var/www/html

  db:
    image: mariadb:10.7
    restart: always
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:
```

## .env

```env
SERVER_PORT=127.0.0.1:8000
```
