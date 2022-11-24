---
title: 'ARK: Survival Evolved Dedicated Server'
date: '2022-07-30 12:00:00'
draft: false
noindex: false
channel: 技術ノート
category: Game
tags:
  - Game
  - 'ARK: Survival Evolved'
  - Docker
---
# ARK: Survival Evolved Dedicated Server

以下のDockerイメージを使う。

- [https://hub.docker.com/r/hermsi/ark-server/](https://hub.docker.com/r/hermsi/ark-server/)

## docker-compose.yml

イメージのタグは適宜最新のものに更新する。

```yaml
version: '3.8'

services:
  ark:
    image: hermsi/ark-server:tools-1.6.61a
    restart: always
    volumes:
      - ./ark-server:/app
      - ./ark-server-backups:/home/steam/ARK-Backups
    environment:
      - SESSION_NAME=${SESSION_NAME}
      - SERVER_MAP=${SERVER_MAP}
      - SERVER_PASSWORD=${SERVER_PASSWORD}
      - ADMIN_PASSWORD=${ADMIN_PASSWORD}
      - MAX_PLAYERS=${MAX_PLAYERS}
      - UPDATE_ON_START=${UPDATE_ON_START}
      - BACKUP_ON_STOP=${BACKUP_ON_STOP}
      - PRE_UPDATE_BACKUP=${PRE_UPDATE_BACKUP}
      - WARN_ON_STOP=${WARN_ON_STOP}
    ports:
      # Port for connections from ARK game client
      - "0.0.0.0:7777:7777/udp"
      # Raw UDP socket port (always Game client port +1)
      - "0.0.0.0:7778:7778/udp"
      # RCON management port
      - "127.0.0.1:27020:27020/tcp"
      # Steam's server-list port
      - "0.0.0.0:27015:27015/udp"
```

## .env

```env
SESSION_NAME=my-ark-session-name
SERVER_MAP=TheIsland
SERVER_PASSWORD=myserverpasword
ADMIN_PASSWORD=myadminpassword
MAX_PLAYERS=10
UPDATE_ON_START=false
BACKUP_ON_STOP=false
PRE_UPDATE_BACKUP=true
WARN_ON_STOP=true
```

## ポート公開

詳細は調査中。

- 7777
- 7778
- 27015

## サーバスペック

おそらく主にはスポーン済みの恐竜の数でメモリ消費量が増える。

テイムした恐竜を拠点周りなどにたくさん出したままにしておくと、メモリ使用量が増えてサーバーが重くなる。

テイム数ほぼ0で物理メモリ2GB程度、テイム数20-30程度で物理メモリ5GB程度。
テイム数（出しっぱなし）もあまり増えすぎると影響しそう。
スポーンシステムに詳しくないのでよくわからないが、主には探索範囲が広がるにつれてスポーン済み恐竜の数が増えて物理メモリ使用量が増えると思われる。

## ゲーム設定

サーバ起動後、`ark-server`ディレクトリには、ほかのゲームファイルと一緒に`GameUserSettings.ini`と`Game.ini`が生成される。

デフォルト設定で遊ぶと、とても時間のかかるゲーム（リアル時間丸一日拘束されたりとかの認識）なので、サーバー利用者と相談して適宜ゲームバランスを調整するのがよい。

設定の変更には、サーバーの再起動が必要なので、再起動時にはサーバー利用者に告知して、安全な場所でログアウトできるように余裕をもって再起動するようにする。

自分のサーバーでは、以下のシングルプレイ用設定をベースに、ゆるめの設定にしている。

- [https://okanezakuzaku.net/post-1291/](https://okanezakuzaku.net/post-1291/)

設定一覧は、以下を参考にするとよい。

- [https://ark.fandom.com/ja/wiki/%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E6%A7%8B%E6%88%90](https://ark.fandom.com/ja/wiki/%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E6%A7%8B%E6%88%90)
- [https://wikiwiki.jp/arkse/%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E8%A8%AD%E5%AE%9A](https://wikiwiki.jp/arkse/%E3%82%B5%E3%83%BC%E3%83%90%E3%83%BC%E8%A8%AD%E5%AE%9A)
- DifficultyOffset: [https://ark.fandom.com/ja/wiki/%E9%9B%A3%E6%98%93%E5%BA%A6](https://ark.fandom.com/ja/wiki/%E9%9B%A3%E6%98%93%E5%BA%A6)
- Lv上限: [https://wikiwiki.jp/arkse-ps4/%E3%82%88%E3%81%8F%E3%81%82%E3%82%8B%E8%B3%AA%E5%95%8F#u4288cfd](https://wikiwiki.jp/arkse-ps4/%E3%82%88%E3%81%8F%E3%81%82%E3%82%8B%E8%B3%AA%E5%95%8F#u4288cfd)

### GameUserSettings.ini

```ini
[ServerSettings]
; (略)
TamingSpeedMultiplier=20.0
serverPVE=True
DifficultyOffset=1.0
OverrideOfficialDifficulty=5.0
ShowFloatingDamageText=True
ResourcesRespawnPeriodMultiplier=0.1
NightTimeSpeedScale=30.0
PlayerCharacterWaterDrainMultiplier=0.3
PlayerCharacterFoodDrainMultiplier=0.3
MaxGateFrameOnSaddles=10
AllowFlyerCarryPvE=True
XPMultiplier=3.0
; (略)
```

### Game.ini

```ini
[/script/shootergame.shootergamemode]
MatingIntervalMultiplier=0.01
EggHatchSpeedMultiplier=60.0
BabyMatureSpeedMultiplier=60.0
BabyImprintAmountMultiplier=60.0
BabyCuddleIntervalMultiplier=0.01
bDisableStructurePlacementCollision=True
bPvEDisableFriendlyFire=True
```

## ゲームバージョンアップデート

調査中。

## ゲームクライアントからの接続

サーバー参加者には、`SESSION_NAME`と`SERVER_PASSWORD`を伝える。

1. ゲーム起動後、「サーバー検索」を開く
2. セッションフィルターを「非公式」に設定
3. 「パスワードありを表示」にチェックを入れる
4. ネームフィルターにセッション名 `SESSION_NAME` を入力
5. 接続時にパスワード `SERVER_PASSWORD` を入力

![ARK スタート画面](images/20220730115252_1.jpg)

![ARK サーバー検索](images/20220730115650_1_mask.jpg)

## RCONによる管理コマンド実行

- RCONのDockerイメージ: [https://hub.docker.com/r/aoirint/rcon](https://hub.docker.com/r/aoirint/rcon)
  - RCONのリポジトリ: [https://github.com/n0la/rcon](https://github.com/n0la/rcon)

### Makefile

```makefile
include .env

.PHONY: list-players
list-players:
	docker run --rm --network host aoirint/rcon:20220806.1 rcon -H 127.0.0.1 -p 27020 -P "${ADMIN_PASSWORD}" --minecraft ListPlayers
```

## ゲーム内での管理コマンド実行

- [https://ark.fandom.com/ja/wiki/Console_Commands](https://ark.fandom.com/ja/wiki/Console_Commands)
- [https://kamigame.jp/ARK/PS4/コマンド.html](https://kamigame.jp/ARK/PS4/コマンド.html)

ゲーム画面でTabキーを押すとコマンドを入力できる。もう1度Tabキーを押すとコマンドのログを確認できる。

コマンドの実行には`ADMIN_PASSWORD`が必要。

```
EnableCheats ADMIN_PASSWORD

SetCheatPlayer true
```

作業後、

```
SetCheatPlayer false
```

EnableCheatsによる管理者への昇格（チャット欄で名前の前に星が付く）は、一度サーバーからログアウトすると元に戻る。

Tabキーで表示されるコマンドのログでパスワードはマスクされることはないので、録画や配信時は注意する。

### TribeIDを調べる

```
SetCheatPlayer true
```

実行後、トライブの建造物や恐竜にフォーカスするとTribeIDが表示される。
