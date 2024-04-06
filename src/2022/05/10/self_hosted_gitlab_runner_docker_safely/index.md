---
# https://scrapbox.io/eyamigusa/Self-hosted_GitLab_Runner%E3%81%A7%E5%AE%89%E5%85%A8%E3%81%ABdocker%E3%82%92%E6%89%B1%E3%81%88%E3%82%8B%E3%82%88%E3%81%86%E3%81%AB%E3%81%99%E3%82%8B
title: Self-hosted GitLab Runnerで安全にdockerを扱えるようにするためにどうしたらいいかわからない
date: '2022-05-10T12:47:51+09:00'
updated: '2022-06-24T10:10:00+09:00'
draft: false
noindex: false
channel: ダイアリー
category: 技術
tags:
  - GitLab
  - Docker
---
# Self-hosted GitLab Runnerで安全にdockerを扱えるようにするためにどうしたらいいかわからない

適当に検索すると、ホストへの特権昇格を許す設定ばかり見かける。
自分だけしか使わないならば構わないかもしれないけれど、なんらかのCIを実行する権限さえあれば、ホストのroot権限をとれるというのは危険だと思う。
（コードレビュー必須でPRではマージ済みのCIが実行されるならいいのか...？）

GitLabの公式Shared Runnerがやっているように、docker-machineを使うのがいいはず。
おそらくdocker入りの仮想マシンをCIジョブごとに立てて、privilegedなコンテナからブレイクアウトしても仮想マシンの外に出られないようになっていると思う。
GitLabはいろいろオープンなので、実際のShared Runnerの構成・設定が公開されている。たいへん助かる。

- [https://docs.gitlab.com/ee/ci/runners/saas/linux_saas_runner.html](https://docs.gitlab.com/ee/ci/runners/saas/linux_saas_runner.html)

のだけれども、GCPでこれを稼働させるのはちょっと費用面がよくわからない...。

---

- ツリー
  - [https://twitter.com/aoirint/status/1523826104343343105](https://twitter.com/aoirint/status/1523826104343343105)
  - [https://twitter.com/aoirint/status/1524426103615500288](https://twitter.com/aoirint/status/1524426103615500288)
