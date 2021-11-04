---
title: 'cron'
date: '2021-11-05 02:00:00'
draft: false
category: Command Utility
tags:
  - cron
---

# cron

Linux上でプログラムの実行をスケジュールするためのパッケージ。

独自の書式をもつcrontabファイルでスケジュールを設定する。

## 想定環境

Ubuntu 20.04

## /etc/crontab

```shell
SHELL=/bin/bash
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=

# m h dom mon dow user  command
0 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
0 0    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
0 0    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
0 0    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```

crontabの時刻形式は独特で、複雑な設定を新しく書き起こすのは面倒くさい。

デフォルトの`/etc/crontab`では、`/etc/cron.hourly`、`/etc/cron.daily`のようなディレクトリの内部の実行ファイルを
定期実行するように設定されているので、ここに雑にシェルスクリプトを入れると簡単にcronを体験できる。

`run-parts`は、指定したディレクトリ内の実行ファイルを、ファイル名でソートした順番で実行する。
以下のように、ファイル名にソートを考慮した番号付けをすることで、ジョブの実行順を制御できる（`10_myjob`→`20_yourjob`→`33_goma`の順で実行される）。

```
10_myjob
20_yourjob
33_goma
```

cronが正しく動作しているか確認するには、`journalctl`を使うとよい。

```shell
journalctl -u cron.service -f -n 20
```

cronにより実行されたプログラムの標準出力や、標準エラー出力を取得するのは手間がかかる。
cronは、プログラムの実行に失敗したときは、ログをメールで送るという動作をするが、メールが送信できるように設定しておかなければならない。
上のcrontabファイルでは、`MAILTO`を空にすることでメールの送信を試みないようにしている（送信できなかったメールが`/var/spool/mail`に蓄積されてストレージを圧迫するため）。

ログを確認するには、プログラム中でファイルに書き出すようにする方法をとることになる。

シェルスクリプトからSlackに通知を送るには、以下のようにするとよい（`sudo apt install curl jq`）。

- <https://api.slack.com/apps>

```shell
#!/bin/bash
WORKDIR=/work
WEBHOOK_URL=https://hooks.slack.com/services/***

cd "$WORKDIR"
ERROR=$(mycommand | tee /dev/stderr)

if [[ $? -ne 0 ]]; then
  SHORT_ERROR=${ERROR:0:1000}
  DATA=$(jq --arg key0 "text" --arg value0 "ERROR: $SHORT_ERROR" '. | .[$key0]=$value0' <<< '{}')
  curl -X POST -H 'Content-type: application/json' --data "$DATA" "$WEBHOOK_URL"
fi
```

## /etc/cron.d

crontab形式の設定を複数のファイルに分けて配置するためのディレクトリ。
例えば、`/etc/crontab`を直接編集せずに、アプリケーションごとにcrontabを配置するようなことができる。
また、実体を別の場所においておいて、シンボリックリンクを`/etc/cron.d`以下に配置することもできる（後述の権限設定に注意）ので、設定のgit管理にも便利（リポジトリ内のMakefileで、`/etc/cron.d/`以下にシンボリックリンクを張るようにしておくなど）。

crontabファイルを一から書くのは慣れが必要なので、上の`/etc/crontab`をコピーし、
もともとの設定を`#`でコメントアウトしておいて、改変しながら使うのが便利だと思う。

なお、crontabファイルの所有者は`root`で、所有者以外に書き込み権限があってはならない（`chmod 644 crontab` または `chmod 600 crontab`）という制約があるので注意。

また、`SHELL`・`PATH`のような設定は、`/etc/crontab`とは独立しているので、改めて記述しなければならない。

例えば、デフォルトでは、`SHELL`は`/bin/sh`であり、`PATH`に`/usr/local/bin`が含まれていない。
`PATH`をそれぞれのcrontabファイルで設定しないと、ターミナルで動かしたときと違って特定のコマンドが動かない、というような問題が起こることがある。
例えば、`docker-compose`は`/usr/local/bin`によくインストールされるので、`docker-compose`コマンドを含むジョブがうまく動かない、ということがある。
