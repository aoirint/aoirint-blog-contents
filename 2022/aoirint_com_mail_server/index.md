---
title: 'aoirint.com 個人用メールサーバ'
date: '2022-12-09T06:35:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: サービス運用ログ
tags:
  - サービス運用ログ
  - サーバー運用
  - メール
---
# aoirint.com メールサーバ

2022-12-09より、aoirint.comにて、個人用メールサーバの運用を始めました。

aoirint.comアドレス宛てにメールが送信できない、受信できない、不正中継されている、セキュリティ上の問題などがあれば、Twitter、Mastodon、Keybaseなどにご連絡ください。

## 更新履歴

- 2022-12-09 運用開始

## 構成

### DNS

DNSでは、MXレコード、SPFレコードを設定しています。


### 外部からのメールを内部に転送するSMTPサーバ mx01.aoirint.com

#### SMTP

|||
|:--|:--|
|暗号化|STARTTLS（必須）|
|認証|なし|
|ホスト名（TLS証明書のコモンネーム）|mx01.aoirint.com|
|ポート番号|25|
|外部へのメール送信|禁止|

#### SMTPS

|||
|:--|:--|
|暗号化|暗黙的TLS|
|認証|なし|
|ホスト名（TLS証明書のコモンネーム）|mx01.aoirint.com|
|ポート番号|465|
|外部へのメール送信|禁止|


### 外部にメールを送信するSMTPサーバ smtp.aoirint.com

#### Submission

|||
|:--|:--|
|暗号化|STARTTLS（必須）|
|認証|平文パスワード|
|ホスト名（TLS証明書のコモンネーム）|smtp.aoirint.com|
|ポート番号|587|
|外部へのメール送信|許可（認証必須）|


### IMAPサーバ imap.aoirint.com

##### IMAPS

|||
|:--|:--|
|暗号化|暗黙的TLS|
|認証|平文パスワード|
|ホスト名（TLS証明書のコモンネーム）|imap.aoirint.com|
|ポート番号|993|


### POP3サーバ pop.aoirint.com

#### POP3S

|||
|:--|:--|
|暗号化|暗黙的TLS|
|認証|平文パスワード|
|ホスト名（TLS証明書のコモンネーム）|pop.aoirint.com|
|ポート番号|995|
