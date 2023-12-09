---
title: 'Ubuntu, ファイルシステムがマウントされていない場合に書き込みを失敗させる'
date: '2023-12-09T17:25:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: 'Ubuntu'
tags:
  - Ubuntu
  - Filesystem
---
# Ubuntu, ファイルシステムがマウントされていない場合に書き込みを失敗させる

外部ストレージなどのファイルシステムのマウントに失敗したとき、
定期実行スクリプトやDockerコンテナなどが、実際にはマウントされていないマウントポイント以下に書き込みしてしまう場合があります。

`/etc/fstab`から`nofail`の指定を外せば、マウントに失敗した場合にOSが起動しなくなることで誤った操作を防げますが、
外部ストレージと関係のない他のサービスが共存している、遠隔操作を前提とした運用をしている、などの理由で、OSは起動してほしい場合があります。

マウントポイントがext4ファイルシステムにある場合、`chattr`コマンドを使って、変更を禁止することができます。

`/mnt/mystorage`のマウントを解除した状態で、以下のように指定します。

```shell
chattr +i /mnt/mystorage
```

これによりマウントしている間、immutable属性がなくなり、変更できるようになります。

```shell
mount /mnt/mystorage
```

- [linux - Prevent the possiblity of writing data to an unmounted mount point directory - Server Fault](https://serverfault.com/questions/570255/prevent-the-possiblity-of-writing-data-to-an-unmounted-mount-point-directory)
- [chattr(1): change file attribs on file system - Linux man page](https://linux.die.net/man/1/chattr)
