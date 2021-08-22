---
title: SSH Port Forwardingを使ってNAT間で通信するPort Proxy Server in Docker
# og_image:
# twitter_card: summary_large_image
# og_description: テンプレート
date: '2021-03-21 03:32:29'
draft: false
category: Network
tags:
  - Network
  - SSH
  - Docker
  - Proxy
---

# SSH Port Forwardingを使ってNAT間で通信するPort Proxy Server in Docker

## SSHポートフォワーディング：踏み台による中継接続

SSHポートフォワーディングには、
ローカルポートフォワーディング、
リモートポートフォワーディングの2種類があります。

- [sshポートフォワーディング - Qiita](https://qiita.com/mechamogera/items/b1bb9130273deb9426f5)

ローカルポートフォワーディングでは、
SSHサーバ側から見えるネットワークポートを
SSHクライアント側に転送することができます。
例えば、ファイアウォール（NAT）に守られたネットワーク（ネットワークA）内にあるWebサーバ（ホストX）に
外部ネットワーク（ネットワークB）にあるクライアント（ホストY）から接続したいとき、
踏み台となる、ネットワークBから接続可能なSSHサーバ（ホストZ）がネットワークA内にあれば、
ホストYがホストZにSSH接続することで、
ホストXのWebサーバのネットワークポートに、
ホストYの指定したポートから接続できるようになります
（「ホスト」は「各ネットワークに接続した、NICに割り当てられたIPアドレス」に対応）。

```shell
user@hostY.networkB.example: $ ssh user@hostZ.networkA.example -L "127.0.0.1:10080:hostX.networkA.example:80"

user@hostY.networkB.example: $ wget http://127.0.0.1:10080/index.html
I am "hostX.networkA.example:80".
```

リモートポートフォワーディングでは、
SSHクライアント側から見えるネットワークポートを
SSHサーバ側に転送することができます。
先の例でいうと、ホストXがネットワークBに接続していて、
ファイアウォール（NAT）によってネットワークAにあるホストZから接続できないとき、
ホストYがホストZにSSH接続することで、ホストZからホストXに接続できるようになります。

```shell
user@hostY.networkB.example: $ ssh user@hostZ.networkA.example -R "127.0.0.1:10080:hostX.networkB.example:80"

user@hostZ.networkA.example: $ wget http://127.0.0.1:10080/index.html
I am "hostX.networkB.example:80".
```

この方法は、ネットワーク内のあるホストにSSH接続ができる環境さえあれば、
VPNや他のTCPプロキシをセットアップするより簡単に隠されたネットワークにアクセスすることができるように思います。



## ポートプロキシサーバ：2種類のSSHポートフォワーディングの組み合わせ

ここで、別の構成のネットワークについても考えてみます。
新しくネットワークCを導入して、
ホストXはNAT内のネットワークC（`hostX.networkC.example`）、
ホストYはNAT内のネットワークB（`hostY.networkB.example`）、
ホストZはネットワークB、Cからともに接続可能なネットワークA（`hostZ.networkA.example`）
という構成を考えてみます。

この構成で、ホストYからホストXに接続可能な環境を作るには、次のような手順が考えられます。

1. ホストXからホストZにSSH接続し、リモートポートフォワーディングによってホストZ上の"ポートQ"をホストX上の"ポートP"に転送する
2. ホストYからホストZにSSH接続し、ローカルポートフォワーディングによってホストY上の"ポートR"をホストZ上の"ポートQ"に転送する

これにより、ホストYは、自身のポートRを使って、ホストXのポートPに接続することができます。
もしホストXに物理的にアクセスできない環境でも、
あらかじめホストXをホストZに常時接続するように設定しておけば、
その接続（ポートQからポートPへの転送）が生きている限り、好きなタイミングでホストYからホストXに接続（ポートRからポートQへの転送、すなわちポートRからポートPへの転送）することができます。



## SSH Port Forwardingを使ったPort Proxy Server in Docker
前項のポートプロキシサーバの問題として、次の3つを考えました。

1. ホストYのユーザは、ホストZ上のローカルポート（ローカルループバックアドレスにバインドしたポート）に自由にアクセスできる
2. ホストYのユーザは、ホストZ上のファイルに（権限の範囲で）自由にアクセスできる
3. ホストZのシステム上にホストYのユーザがログインするためのユーザを用意しなければならない

これらの問題の解決法として、Dockerを使ってホストZ上のSSHサーバ・ユーザ管理を仮想化することを試してみました。

- [Docker の bridge と host ネットワークについて勉強する - Qiita](https://qiita.com/toshihirock/items/f5b9f7799ec8bf8c328e)

作ったシステムはGitHubにおいています。

- ホストZ用のシステム：https://github.com/aoirint/SSHPortForwardingProxy
- ホストX用のシステム：https://github.com/aoirint/SSHPortForwardingProxyClient
- ホストY：通常の`ssh`コマンドを使用

仮想化の不完全な点として、ホストX用のシステムについて、設定が煩雑になるのを避けるため、`network_mode: host`（`--net host`）を指定してしまっています。
また、パスワードログインやログインシェルは無効化していますが、まだSSHサーバの設定（アクセス制限）に不備があるかもしれません。
他に、ホストXからホストZ上の仮想SSHサーバへの初回接続でフィンガープリントの確認をスキップしています。


### 事前準備の手順
認証はすべて公開鍵認証を使うことを想定します。
そのため、あらかじめ各ホスト間で公開鍵の共有が必要です。

1. [ホストX] SSH鍵を生成し、公開鍵をホストZに転送する
2. [ホストY] SSH鍵を生成し、公開鍵をホストZに転送する
3. [ホストY] SSH鍵を生成し、公開鍵をホストXに転送する

### 準備手順
1. [ホストZ] `git clone https://github.com/aoirint/SSHPortForwardingProxy.git`
2. [ホストZ] `docker-compose.override.yml.template`を`docker-compose.override.yml`にコピーする
3. [ホストZ] どのポートで仮想SSHサーバをホストするか決め、`docker-compose.override.yml`にポートマッピングを記述する
    - ここでは、ホストX、Yから仮想SSHサーバに`hostZ.networkA.example:10080`でアクセスすることとする
    - 仮想サーバ（VirtualZ）は、DockerによってネットワークVに分離される（ブリッジ）。SSHサーバは、`virtualZ.networkV.example:22`でホストすることとする
4. [ホストZ] ホストX、ホストYの公開鍵を`/authorized_keys/USERNAME`に通常の`authorized_keys`と同様の形で格納する
    - USERNAMEは仮想SSHサーバに各ホストからログインするときのユーザ名として使われる
    - 複数のファイルを格納した場合、それぞれのユーザが作られる
    - ここでは、ホストX用のユーザをuserX、ホストY用のユーザをuserYとする
5. [ホストZ] `docker-compose up -d`
6. [ホストX] `git clone https://github.com/aoirint/SSHPortForwardingProxyClient.git`
7. [ホストX] `docker-compose.override.yml.template`を`docker-compose.override.yml`にコピーする
8. [ホストX] ホストX上のどのポートを仮想サーバ上のどのポートに転送するか決め、`docker-compose.override.yml`に設定を記述する
    - ここでは、ホストXのSSHサーバ`hostX.networkC.example:22`を`virtualZ.networkV.example:10022`に転送することとする
    - 秘密鍵のマウントおよびコンテナ内でのパスを合わせて設定する
9. [ホストX] `docker-compose up -d`

### 使用手順
1. [ホストY] `ssh userY@hostZ.networkA.example -p 10080 -N -i "VirtualZへの秘密鍵のパス" -L "127.0.0.1:20080:127.0.0.1:10022"`
    - 接続をバックグランドで維持しておく
    - ホストXのSSHサーバにホストYの20080番ポートからアクセスできるようになった（ここではホストY内のローカルアクセスに限定）
2. [ホストY] `ssh userYinHostX@localhost -p 20080 -i "ホストXへの秘密鍵のパス"`
    - ホストYからホストXにSSH接続ができる（目的達成）


## 未解決の問題
- ホストYは、ネットワークAから見えるホストに（ホストZのアドレスで）自由に接続できる
    - `--cap-add​=NET_ADMIN`と`iptables`を使って制限できるかもしれない
