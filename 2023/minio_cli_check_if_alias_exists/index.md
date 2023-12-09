---
title: 'MinIO CLI（mcコマンド）でエイリアスが登録されているか確認する'
date: '2023-12-09T17:05:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: 'Object Storage'
tags:
  - 'Object Storage'
  - MinIO
  - 'MinIO CLI'
---
# MinIO CLI（mcコマンド）でエイリアスが登録されているか確認する

```shell
$ mc --version
mc version RELEASE.2023-12-02T11-24-10Z (commit-id=d920e2b34b22a15bca4cd081201d3b301c623d87)
Runtime: go1.21.5 linux/amd64
Copyright (c) 2015-2023 MinIO, Inc.
License GNU AGPLv3 <https://www.gnu.org/licenses/agpl-3.0.html>
```

MinIO CLI（mcコマンド）では、サーバーURLや認証情報などを`mc alias set`コマンドで保存して、
設定したエイリアスを使って操作します。

`mc cp`や`mc mirror`では、`{alias}/{bucket}/{object}`のように操作対象のオブジェクトを指定します。
このとき`alias`が登録されていなかった場合、ファイルシステム上の相対パスとして扱われるため、意図しない操作が行われるおそれがあります。

エイリアス`myalias`が登録されているかどうかは、`mc alias list {alias}`コマンドの終了コードで判別できます。
終了コードが0の場合、エイリアスは存在します。終了コードが1の場合、エイリアスは存在しません。

```shell
mc alias list myalias

echo $?
```

シェルスクリプトでは、以下のようなコードを追加して、エイリアスが存在しなかった場合にスクリプトを異常終了させられます。

```shell
# set +e

# check alias exists
mc alias list myalias >/dev/null 2>/dev/null
if [ $? != 0 ]; then
    echo "Error: mc alias myalias does not exist" > /dev/stderr
    exit 1
fi

# set -e
```
