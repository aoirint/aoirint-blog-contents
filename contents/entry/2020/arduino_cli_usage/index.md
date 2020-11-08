---
canonical_url: ./
title: arduino-cliの使い方
# og_image:
# twitter_card: summary_large_image
og_description: arduino-cliの使い方
date: '2020-11-09 08:00:00'
draft: false
category: スニペット
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

[Arduino CLIの公式ドキュメント](https://arduino.github.io/arduino-cli/latest/)に従ってインストールする。後述する`screen`コマンドも合わせてインストールする。
`arduino-cli`はDockerイメージも配布されているのでお好みで。

```bash
curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | BINDIR=~/.local/bin sh
echo "export PATH=\"\$PATH:\$HOME/.local/bin\"" >> ~/.bashrc
source ~/.bashrc

sudo apt install screen
```

ボード一覧を更新しておく。

```bash
arduino-cli core update-index
```


## スケッチの作成
特に特殊なファイルを必要としたりはしないので、
Arduino IDEで作成しても、好きなテキストエディタで作成してもよい。

一応テンプレート付きのスケッチを作成するには、`arduino-cli sketch new SKETCH_NAME`を実行する。

以下、`arduino-cli`のコマンドはスケッチのディレクトリで実行する。


## FQBNの確認
`arduino-cli`でボードを扱うときには、ボード名にあたる`FQBN`というコロンで区切られた文字列を使う。
（対応しているボードならば？）`arduino-cli board list`コマンドでPCに接続しているボードのFQBNを調べられる。

- Arduino UNO: `arduino:avr:uno`
- ESP32-DevKitC: `esp32:esp32:esp32`


## ボード情報のインストール

コンパイルには別途ボード情報をインストールする必要があり、以下のようにコマンドを実行する。

```bash
# Arduino (AVR)
arduino-cli core install arduino:avr

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

ボードにコンパイルしたスケッチを書き込む。
シリアルポートを`-p`オプションで指定する。

```bash
arduino-cli upload -b arduino:avr:uno -p /dev/ttyACM0
```

コンパイル成果物はデフォルトで`./build`ディレクトリに書き出されるので、
`.gitignore`などに追加しておく。


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
