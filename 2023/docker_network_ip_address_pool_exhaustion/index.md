---
title: '自宅サーバのDocker NetworkのIPアドレスプールが枯渇した'
date: '2023-01-26T01:30:00+09:00'
draft: false
noindex: false
channel: 技術ノート
category: Docker
tags:
  - Docker
  - Network
---
# 自宅サーバのDocker NetworkのIPアドレスプールが枯渇した

- [https://twitter.com/aoirint/status/1617993934625722368](https://twitter.com/aoirint/status/1617993934625722368)

弊自宅サーバでは、サーバソフトウェアや定期実行プログラムのデプロイに、基本的にDocker Composeを使用しています。

先日、新しいソフトウェアをDocker Composeでデプロイしようとして、以下のエラーが発生しました。

```plain
Error response from daemon: could not find an available, non-overlapping IPv4 address pool among the defaults to assign to the network
```

Docker Networkが使用するIPアドレスプールが枯渇し、新しいIPアドレスブロックを確保できなかったという内容のエラーです。

## 環境

- Ubuntu 20.04

```shell
$ docker -v
Docker version 20.10.23, build 7155243

$ docker compose version
Docker Compose version v2.15.1
```

## 原因

このエラーは、30個程度のDocker Networkを作成すると発生するようです。

Docker Composeでは通常、プロジェクト1つあたり、1つのDocker Networkが作成されます（複雑なプロジェクトでは、2つ以上使用することがあるかもしれません）。
手元の環境では、Docker Composeのプロジェクトが27個稼働しており、28個目のプロジェクトを起動したときにエラーが発生しました。

Docker Networkの一覧は、以下のコマンドで確認できます。

```shell
docker network ls
```

Docker NetworkのIPアドレスには、いわゆるプライベートIPアドレスの範囲が使用されます。

プライベートIPアドレスの範囲は、以下のようになります（CIDR表記）。

- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

作成されたDocker Networkに割り当てられたIPレンジは`docker network inspect`コマンドで確認できます。
以下の記事で紹介されているコマンドが便利です。

- [https://qiita.com/tksugimoto/items/e42b0d6931898671734f#各ネットワークのipレンジ](https://qiita.com/tksugimoto/items/e42b0d6931898671734f#%E5%90%84%E3%83%8D%E3%83%83%E3%83%88%E3%83%AF%E3%83%BC%E3%82%AF%E3%81%AEip%E3%83%AC%E3%83%B3%E3%82%B8)

```shell
sudo docker network inspect $(sudo docker network ls -q) | grep -E "Subnet|Name"
```

以下のIssueコメントおよびソースコードを参照すると、Docker Networkに使用できるサブネットの数がわかります。
なお、Docker Networkにはスコープという概念があり、通常は`local`です。Docker Swarmを使用している場合、`global`が使われるようですが、詳しくないため、ここでは`local`のみ扱います。

- [https://github.com/docker/docs/issues/8663#issuecomment-956438889](https://github.com/docker/docs/issues/8663#issuecomment-956438889)
  - [https://github.com/moby/libnetwork/blob/05b93e0d3a95952f70c113b0bc5bdb538d7afdd7/ipamutils/utils.go#L18-L20](https://github.com/moby/libnetwork/blob/05b93e0d3a95952f70c113b0bc5bdb538d7afdd7/ipamutils/utils.go#L18-L20)

|IPレンジ|ネットワーク部のサイズ|確保可能なサブネットの数|
|:--|:--|:--|
|172.17.0.0/16|/16|1|
|172.18.0.0/16|/16|1|
|172.19.0.0/16|/16|1|
|172.20.0.0/14|/16|4|
|172.24.0.0/14|/16|4|
|172.28.0.0/14|/16|4|
|192.168.0.0/16|/20|16|

つまり、スコープ`local`に使用されるサブネットの数は31個です。

ただし、Ethernet NICが家庭用ルータに接続されたマシンであれば、NAT/NAPTのため、
プライベートIPアドレスが割り当てられているでしょう。

また、Dockerのインストール時には、コンテナのネットワークとしてデフォルトで使用される`docker0`というブリッジネットワークが作成され、
プライベートIPアドレスが割り当てられます。

また、libvirtがインストールされた環境であれば、`virbr0`、`virbr1`などの仮想NICが作成され、
プライベートIPアドレスが割り当てられます。
libvirtが管理するネットワークのリストは、`virsh net-list`で確認できます。

手元の環境では、上記により4個のサブネットが使用済みであり、当初の31個から引いて、Docker Networkの作成可能個数は27個となります。

使用されていないDocker Networkは、`docker network prune`コマンドで削除できますが、すべてのDocker Networkが使用中であり、
このコマンドでは1つも削除することができませんでした。

## 解決

Docker Networkに使用するIPアドレスの範囲は、`/etc/docker/daemon.json`に`default-address-pools`を設定することで上書きできます。
適用にはDockerデーモンの再起動が必要です。また、IPアドレスを割り当て直すため、ネットワークを作り直す必要もあります。

今回は、以下の記事のようにサブネットのホスト部を縮小し、確保可能なサブネットの数を増やすアプローチをとりました。
また、`172.16.0.0/12`があれば十分そうだったため、いったん`192.168.0.0/16`は、家庭内ネットワークのために予約することにしました。

- [https://tech.actindi.net/2019/03/14/170000](https://tech.actindi.net/2019/03/14/170000)

`/etc/docker/daemon.json`に以下の項目を追加しました。

```json
"default-address-pools": [
  {
    "base": "172.16.0.0/12",
    "size": 24
  }
]
```

|IPレンジ|ネットワーク部のサイズ|確保可能なサブネットの数|
|:--|:--|:--|
|172.16.0.0/12|/24|4096|

ネットワーク部24ビットのうち、先頭12ビットを固定しているので、残りの12ビットで4096個のサブネットが作れます。
1つのサブネットあたり、256個のIPアドレスが確保されます。
実際には、libvirtとの競合などにより、すべてのサブネットが利用できるわけではなさそうですが、それでも十分な数が利用できると思われます。

4096個のプロジェクトともなると、CPUパワーやメモリ量、またDocker Composeで管理しきれるのか、という方が先に課題になるでしょう。

今回は自宅サーバということで、これらに多少の余裕があり、また、利用者が1から数人の小規模サービスがいくつも稼働している形態のため、27個で足りなくなったというわけでした。
