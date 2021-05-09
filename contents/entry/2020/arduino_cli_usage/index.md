---
canonical_url: ./
title: arduino-cliの使い方
# og_image:
# twitter_card: summary_large_image
og_description: arduino-cliの使い方
date: '2020-11-09 08:00:00'
updated: '2021-05-09 17:30:00'
draft: false
category: Arduino
tags:
- Arduino
- 'arduino-cli'
---

# arduino-cliの使い方

```
$ arduino-cli version
arduino-cli Version: 0.13.0 Commit: 693a045
```


## インストール
- [Installation - Arduino CLI](https://arduino.github.io/arduino-cli/latest/installation/)
- [arduino/arduino-cli](https://hub.docker.com/r/arduino/arduino-cli)

[Arduino CLIの公式ドキュメント](https://arduino.github.io/arduino-cli/latest/)に従ってインストールする。後述する`screen`コマンドも合わせてインストールする。
`arduino-cli`はDockerイメージも配布されているのでお好みで。

```bash
curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | BINDIR=~/.local/bin sh
echo "export PATH=\"\$PATH:\$HOME/.local/bin\"" >> ~/.bashrc
source ~/.bashrc

sudo apt install screen
```

プラットフォーム一覧を更新しておく。

```bash
arduino-cli core update-index
```

シリアルポートを使うため、`dialout`グループにユーザを追加する（`/dev/ttyACM#`や`/dev/ttyUSB#`のグループは`dialout`）。

```shell
sudo adduser $USER dialout
```

## スケッチの作成
特に特殊なファイルを必要としたりはしないので、
Arduino IDEで作成しても、好きなテキストエディタで作成してもよい。

一応テンプレート付きのスケッチを作成するには、`arduino-cli sketch new SKETCH_NAME`を実行する。

手動で作成する場合にはArduino IDEと同様、
スケッチ名（ディレクトリ名）と同じ名前で、`SKETCH_NAME.ino`のように
メインのソースコードを作成する。

以下、`arduino-cli`のコマンドはスケッチのディレクトリで実行する。


## FQBNの確認
`arduino-cli`でボードを扱うときには、ボード名にあたる`FQBN`というコロンで区切られた文字列を使う。

`arduino-cli board listall`コマンドで
`arduino-cli`が対応しているボードのFQBN一覧が出力される。
`arduino-cli board listall esp32`のようにキーワードを追加して、
絞り込むこともできる。

また、`arduino-cli board list`コマンドでPCに接続しているボードのFQBNを調べられる場合がある（Arduino系ボードの場合？）。

- Arduino UNO: `arduino:avr:uno`
- Arduino Nano: `arduino:avr:nano`
- Arduino Nano Every: `arduino:megaavr:nona4809
- ESP32-DevKitC: `esp32:esp32:esp32`


## プラットフォームコアのインストール

コンパイルには別途ボードに対応するビルドツールセット（プラットフォームコア）をインストールする必要があり、FQBNから以下のようにコマンドを実行する。

```bash
# Arduino (AVR)
arduino-cli core install arduino:avr

# Arduino Nano Every ほか
arduino-cli core install arduino:megaavr

# ESP32
arduino-cli core install esp32:esp32
```


## ライブラリのインストール

ライブラリの導入には、`arduino-cli lib`コマンドを使う。
例えば、`ArduinoJson`ライブラリを導入するときは、
以下のように検索コマンドで該当するライブラリの名前を確かめ、インストールコマンドを実行する。

```bash
arduino-cli lib search ArduinoJson

arduino-cli lib install ArduinoJson
```


## スケッチのコンパイル

ボードに書き込む前に、まずスケッチ（ソースコード）をコンパイルする。

Arduino UNO（FQBN：`arduino:avr:uno`）に書き込むためのスケッチをコンパイルする。

```bash
arduino-cli compile -b arduino:avr:uno
```

コンパイル成果物はデフォルトで`./build`ディレクトリに書き出されるので、
`.gitignore`などに追加しておく。


Arduino Nano EveryはATmega4809を搭載しているが、デフォルトでArduino UnoやNanoのATmega328Pをエミュレートする互換モードになる。
違いはよくわからないが、低レベルAPIを直接使っている場合に影響があるかもしれない。

```shell
# 互換モード（デフォルト）
arduino-cli compile -b arduino:megaavr:nona4809

# 通常モード
arduino-cli compile -b arduino:megaavr:nona4809:mode=off
```

```
$ arduino-cli board details arduino:megaavr:nona4809
Board name:                Arduino Nano Every                                                            
FQBN:                      arduino:megaavr:nona4809                                                      
Board version:             1.8.7                                                                         

Official Arduino board:    ✔                                                                             

Identification properties: VID:0x2341 PID:0x0058                                                         

Package name:              arduino                                                                       
Package maintainer:        Arduino                                                                       
Package URL:               https://downloads.arduino.cc/packages/package_index.json                      
Package website:           http://www.arduino.cc/                                                        
Package online help:       http://www.arduino.cc/en/Reference/HomePage                                   

Platform name:             Arduino megaAVR Boards                                                        
Platform category:         Arduino                                                                       
Platform architecture:     megaavr                                                                       
Platform URL:              http://downloads.arduino.cc/cores/core-ArduinoCore-megaavr-1.8.7.tar.bz2      
Platform file name:        core-ArduinoCore-megaavr-1.8.7.tar.bz2                                        
Platform size (bytes):     875098                                                                        
Platform checksum:         SHA-256:24853e59bfcfcfa09d7ab51011b65f2246e082228b1f14fdaa4cbb2c6aae23b4      

Required tool:             arduino:avr-gcc                                                                                                    7.3.0-atmel3.6.1-arduino5

Required tool:             arduino:avrdude                                                                                                    6.3.0-arduino17          

Required tool:             arduino:arduinoOTA                                                                                                 1.3.0                    

Option:                    Registers emulation                                                                                                mode                     
                           ATMEGA328                                                                      ✔                                   mode=on                  
                           None (ATMEGA4809)                                                                                                  mode=off                 
Programmers:               Id                                                                             Name                               
                           medbg                                                                          Onboard Atmel mEDBG (UNO WiFi Rev2)
```

## スケッチの書き込み

ボードにコンパイルしたスケッチを書き込む。
シリアルポートを`-p`オプションで指定する。

```bash
arduino-cli upload -b arduino:avr:uno -p /dev/ttyACM0
```


## シリアルモニタ

`arduino-cli`にはシリアルモニタ機能がない（追加される予定もない）ので、
ほかのツールを使う必要がある。

- [Feature Request: Serial Monitor · Issue #876 · arduino/arduino-cli](https://github.com/arduino/arduino-cli/issues/876)
    - 機能提案はリジェクトされている（別のツールを使ってね）

`screen`コマンドがよく使われるようだったので、これを使う。
Ubuntuの場合`apt install screen`で導入できる。
/dev/ttyACM0の部分にはシリアルポート名を、115200の部分にはbaudrateを入れる。

```bash
screen /dev/ttyACM0 115200
```

閉じるには`Ctrl+a k`を入力する。


## プロキシ設定
- ["board list" command doesn't follow proxy configuration · Issue #660 · arduino/arduino-cli](https://github.com/arduino/arduino-cli/issues/660)
- [how to use net proxy? · Issue #271 · arduino/arduino-pro-ide](https://github.com/arduino/arduino-pro-ide/issues/271)

`arduino-cli`の通信にプロキシを設定するには、
コンフィグファイルを作成する必要がある。

`arduino-cli config init`を実行すると
`arduino-cli`の設定関係ディレクトリに`arduino-cli.yml`が生成される。
私の環境では、`~/.arduino15/arduino-cli.yml`が生成された。
これを開き、以下のようにプロキシ設定を追加する。

```yaml
board_manager:
  additional_urls: []

（略）

network:
  proxy: http://proxy:port
```


## 関連
- [[Arduino] arduino-cli初期設定(内蔵LEDでLチカ) - Life with IT](https://l-w-i.net/t/arduino/cli_001.txt)
