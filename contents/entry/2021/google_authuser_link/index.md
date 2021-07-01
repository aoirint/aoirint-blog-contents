---
canonical_url: ./
title: GoogleサービスのURLでユーザアカウントを指定する（?authuser=email）
date: '2021-07-01 13:00:00'
draft: false
category: Google
tags:
  - Google
---

# GoogleサービスのURLでユーザアカウントを指定する

Googleサービスでは、URL中の`/u/0/`や`/u/1/`の数値部分を書き換えることで、ログイン順に基づいてユーザの切り替えができる。しかし、ログイン順の異なる端末間でURLをブックマークしたい場合などに不便である。

メールアドレスを指定してユーザを切り替えるには、
URLのGETパラメータに`authuser=mail@example.com`のようにGoogleアカウントのメールアドレスを記述する。

## Google Classroom
- https://classroom.google.com/c/CLASS_ID?authuser=mail@example.com

## Gmail
- https://mail.google.com/?authuser=mail@example.com
