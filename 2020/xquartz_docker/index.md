---
title: Docker Desktop for Mac上のX ClientをホストのXQuartz（X Window Server）で表示する
# og_image:
# twitter_card: summary_large_image
og_description: Docker Desktop for Mac上のX ClientをホストのXQuartz（X Window Server）で表示する
date: '2020-12-20 00:30:00'
draft: false
category: Docker
tags:
  - Docker
  - macOS
---

# Docker Desktop for Mac上のX ClientをホストのXQuartz（X Window Server）で表示する

```
$ docker -v
Docker version 20.10.0, build 7287ab3

$ brew -v
Homebrew 2.6.2
Homebrew/homebrew-core (git revision ce927; last commit 2020-12-19)
Homebrew/homebrew-cask (git revision eb977; last commit 2020-12-19)

$ brew info xquartz
xquartz: 2.7.11 (auto_updates)
https://www.xquartz.org/
/usr/local/Caskroom/xquartz/2.7.11 (74.6MB)
From: https://github.com/Homebrew/homebrew-cask/blob/HEAD/Casks/xquartz.rb
==> Name
XQuartz
==> Description
Open-source version of the X.Org X Window System
```

- Docker Desktop for Mac 3.0.2 (50996)
- macOS Catalina Version 10.15.7


## XQuartzのインストール（HomebrewとHomebrew Cask）
現在は`brew cask`コマンドは非推奨で、`brew`だけでOK（あるいは`--cask`オプションをつける）。
XQuartzの場合は`--cask`をつけなくても内部で勝手に`brew cask`としてインストールしてくれた。
Homebrew CaskというのはGUIアプリケーション向けのHomebrewの拡張らしいが、Homebrewと何が違うのかわからん。

```
Warning: Calling brew cask install is deprecated! Use brew install [--cask] instead.
```

- [The Missing Package Manager for macOS (or Linux) — Homebrew](https://brew.sh/)
- [homebrew-cask — Homebrew Formulae](https://formulae.brew.sh/cask/)
- [Homebrew/homebrew-cask: 🍻 A CLI workflow for the administration of macOS applications distributed as binaries](https://github.com/Homebrew/homebrew-cask)
- [command line - What is the difference between `brew` and `brew cask`? - Ask Different](https://apple.stackexchange.com/questions/125468/what-is-the-difference-between-brew-and-brew-cask)
- [homebrew-cask/USAGE.md at master · Homebrew/homebrew-cask](https://github.com/Homebrew/homebrew-cask/blob/master/USAGE.md)

Homebrewは、開発元からソースコードが配布されていて、そのコンパイル済みのバイナリ（またはソースダウンロード＋自動ローカルビルド）を提供するもので、
Homebrew Caskは、`*.dmg`が配布されていてマウントして`*.app`を`/Applications`にコピーする操作（実際には`/usr/local/Caskroom`にインストールする）のを自動化する、というものなのだろうか?
`--cask`を明示するのは両方に登録されていてもCaskを優先するみたいな指定なのか? XQuartzの場合は`--cask`を付けなくてもCaskとしてインストールされた。

```bash
# brew install xquartz # これでもよさそう
brew install --cask xquartz
```

XQuartz.appを起動する。
シェルに環境変数DISPLAYが設定される（`/private/tmp/com.apple.launchd.***/org.macosforge.xquartz:0`のような値）。

XQuartzと合わせて導入されるxhostやxeyesにパスが通っていないので、`/usr/X11/bin`にパスを通しておく。

```bash
# bashの場合：~/.bash_profile
export PATH=/usr/X11/bin:$PATH
```

起動確認。

```bash
xeyes
```

また、XQuartzのXQuartz > Preferences > Security > Allow connections from network clientsにチェックを入れる。
Dockerからアクセスするために必要。
また、この設定はXQuartzを再起動しないと反映されないので、一度XQuartzをQuitして起動しなおす。


## テスト用Dockerイメージ
```dockerfile
FROM alpine:3

RUN apk --no-cache add xeyes

CMD ["/usr/bin/xeyes"]
```

```bash
docker build . -t xeyes
```

## 確認：Docker Desktop for MacのDNS設定
以下のようなdocker pullに失敗する事象のため、Docker daemonのDNS設定を変更していた。
具体的には、Docker Desktop for Macのタスクバーアイコン > Preferences > Docker EngineのJSON設定欄に
`"dns": [ "primary dns address", "secondary dns adderss" ]`のように設定を書き足して解決していた。

```
failed to solve with frontend dockerfile.v0:
failed to create LLB definition:
failed to authorize:
rpc error: code = Unknown desc = failed to fetch oauth token:
Get https://auth.docker.io/token?scope=repository%3Alibrary%2Fubuntu%3Apull&service=registry.docker.io:
dial tcp 54.236.131.166:443: i/o timeout
```

- [How do I configure which DNS server docker uses in Docker Desktop for Mac? - Stack Overflow](https://stackoverflow.com/questions/44410259/how-do-i-configure-which-dns-server-docker-uses-in-docker-desktop-for-mac)
- [docker run hello-world results in i/o timeout · Issue #1346 · docker/for-mac](https://github.com/docker/for-mac/issues/1346)
- [dns - How to solve i/o timeout error in docker pull - Stack Overflow](https://stackoverflow.com/questions/48042184/how-to-solve-i-o-timeout-error-in-docker-pull)

この状態では、以下のhost.docker.internalを使う方法・hostnameを使う方法が動作せず、この設定は削除する必要があった。
なお最初に起きていた事象は、ネットワークのファイアウォールによってOP53B（DNSブロック）されていたのが原因と思われるが、
この設定を削除したとき、同様のネットワークで事象は復活しなかった（ネットワークの切り替え直後だったために起きていた一時的な問題？）。


## host.docker.internalを使う方法（旧 docker.for.mac.localhost, docker.for.mac.host.internal）
- [How to show X11 windows with Docker on Mac | by Marc Reichelt | Medium](https://medium.com/@mreichelt/how-to-show-x11-windows-within-docker-on-mac-50759f4b65cb)
- [How to display a gui app in a Docker container in macOS | Alessandro Chimetto](http://www.achimetto.me/docker-gui-app-on-macos.html)
- [Mac+dockerでx11アプリケーションを起動する - livaの雑記帳](http://raphine.hatenablog.com/entry/2018/08/14/004634)
- [X11 in docker on macOS へのコメント](https://gist.github.com/cschiewek/246a244ba23da8b9f0e7b11a68bf3285#gistcomment-3477013)
    - [Docker X11 macOS](https://gist.github.com/paul-krohn/e45f96181b1cf5e536325d1bdee6c949)

この方法がシンプルで安全なように思われた。
以下は、X Serverのすべてのアクセス制限を復活させたのち、localhostからのアクセスのみを許可した状態でxeyesをDocker上で起動する。

```bash
xhost -
xhost + localhost

docker run --rm -e DISPLAY=host.docker.internal:0 xeyes
```


## hostnameを使う＋~/.Xauthorityを共有する方法
- [Docker for Mac で X11 アプリケーションを動かす - Qiita](https://qiita.com/hoto17296/items/bdb2ab24bc32b6b7f360)
- [macOS で Docker 内で動かした X11 アプリを表示させる - Qiita](https://qiita.com/kawaz/items/6cf04f923ebfac45a997)

## hostnameを使う＋/tmp/.X11-unixを共有する方法
- [X11 in docker on macOS](https://gist.github.com/cschiewek/246a244ba23da8b9f0e7b11a68bf3285)

## プライベートIPを使う方法
- [Mac DockerでGUIを使いたい（XQuartz版） - あよなの足跡](https://gokids.hatenablog.com/entry/2018/11/14/190000)
- [macos - Running GUI apps on docker container with a MacBookPro host - Stack Overflow](https://stackoverflow.com/questions/37523980/running-gui-apps-on-docker-container-with-a-macbookpro-host)
- [Running GUI applications using Docker for Mac - Sourabh](https://sourabhbajaj.com/blog/2017/02/07/gui-applications-docker-mac/)
- [X11 in docker on macOS へのコメント](https://gist.github.com/cschiewek/246a244ba23da8b9f0e7b11a68bf3285#gistcomment-3119974)

## socatを使う方法（UnixソケットをTCPにリレーする方法）
- [Dockerで稼働するGUIアプリをMacOSXから利用する](https://gist.github.com/asufana/229cdac01fccee1a7d32ca8b5d7cfee6)
- [macos - Xt error: Can't open display, if using default DISPLAY - Stack Overflow](https://stackoverflow.com/questions/37826094/xt-error-cant-open-display-if-using-default-display)

## SSH X11 Forwardingを使う方法
- [dockerコンテナの中で立ち上げたGUIアプリをmacに表示してみる - Qiita](https://qiita.com/machisuke/items/84626eba60ab76d8fc4e)
