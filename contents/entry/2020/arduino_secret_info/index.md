---
canonical_url: ./
title: ソース公開するArduinoプログラムに秘密情報を埋め込む
# og_image:
# twitter_card: summary_large_image
og_description: ソース公開するArduinoプログラムに秘密情報を埋め込む
date: '2020-11-09 07:30:00'
draft: false
category: スニペット
tags:
  - Arduino
---

# ソース公開するArduinoプログラムに秘密情報を埋め込む

[aoirint/RoomSystemSensorESP32: ESP32とFirebaseを使った部屋センシング・オンライン化 クライアント](https://github.com/aoirint/RoomSystemSensorESP32)の開発中に
Arduinoプログラム（`.ino`）にWiFiパスワード・APIキーなどの秘密情報を埋め込む必要が出てきた。

ここでは、Arduino IDEの主要機能を持つCLIソフトウェア`arduino-cli`を使う。

[arduino/arduino-cli: Arduino command line interface](https://github.com/arduino/arduino-cli)


## arduino-cliの使い方
`arduino-cli`でボードを扱うときには、ボード名にあたる`FQBN`というコロンで区切られた文字列を使う。
Arduino系のボードならば`arduino-cli board list`コマンドでPCに接続しているボードのFQBNを調べられる。

コンパイルには別途ボード情報をインストールする必要があり、以下のようにコマンドを実行する。

```bash
arduino-cli core install arduino:avr
```

ライブラリの導入には、`arduino-cli lib`コマンドを使う。
例えば、`ArduinoJson`ライブラリを導入するときは、
以下のように検索コマンドで該当するライブラリの名前を確かめ、インストールコマンドを実行する。

```bash
arduino-cli lib search ArduinoJson

arduino-cli lib install ArduinoJson
```

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

`arduino-cli`にはシリアルモニタ機能がない（追加される予定もない）ので、
ほかのツールを使う必要がある。

- [Feature Request: Serial Monitor · Issue #876 · arduino/arduino-cli](https://github.com/arduino/arduino-cli/issues/876)
    - 機能提案はリジェクトされている（別のツールを使ってね）

screenコマンドがよく使われるようだったので、これを使う。
Ubuntuの場合`apt install screen`で導入できる。
/dev/ttyACM0の部分にはシリアルポート名を、115200の部分にはbaudrateを入れる。

```bash
screen /dev/ttyACM0 115200
```


## 秘密情報の埋め込み

さて、本題に移る。
秘密情報の埋め込みには、以下のようなシェルスクリプト`compile.sh`を作成するのが楽でよい。
`DEFINES=`の部分の`-D`から`=`までの文字列が定数名、`=`の右辺が定数値として
定義された状態でソースコードがコンパイルされる。
ここでは同ディレクトリの`.env`ファイルを読み込んで使用する。
`.env`ファイルのフォーマットはよくあるものと同じで、
改行で区切られ、`#`から始まる行を無視する`KEY=VALUE`形式のテキストファイル。

ボードへの書き込みには以下の`upload.sh`のようなスクリプトを使うとよい。

`screen`コマンドをラップするスクリプト`serialmon.sh`もおいておく。


### compile.sh

```bash
#!/bin/bash

set -eu

if [ -f .env ]; then
    echo "Found .env file."
    export $(cat .env | sed 's/#.*//g' | xargs)
fi


# ESP32-DevKitC
FQBN="esp32:esp32:esp32"

# Arduino UNO
# FQBN="arduino:avr:uno"


DEFINES="-DSECRET_WIFI_SSID=$WIFI_SSID"
DEFINES="${DEFINES} -DSECRET_WIFI_PW=$WIFI_PW"
DEFINES="${DEFINES} -DSECRET_FIREBASE_HOST=$FIREBASE_HOST"
DEFINES="${DEFINES} -DSECRET_FIREBASE_AUTH=$FIREBASE_AUTH"


SKETCH="$(basename $PWD).ino"

arduino-cli compile \
    -b "$FQBN" \
    --build-properties \
        "build.defines=${DEFINES}" \
    "$SKETCH" "$@"
```

プログラム側では以下のようにする。`#x`はコメントではないので注意（文字列リテラルとして展開するマクロ）。

```c
#define _Q(x) #x
#define Q(x) _Q(x)

#ifdef SECRET_WIFI_SSID
  #define WIFI_SSID Q(SECRET_WIFI_SSID)
#else
  #define WIFI_SSID "WIFI-SSID"
#endif

#ifdef SECRET_WIFI_PW
  #define WIFI_PW Q(SECRET_WIFI_PW)
#else
  #define WIFI_PW "WIFI-PASSWORD"
#endif

void initWiFi() {
  WiFi.begin(WIFI_SSID, WIFI_PW);
  Serial.print("WiFi ");
  Serial.print(WIFI_SSID);
  Serial.println(" connecting");

  while (WiFi.status() != WL_CONNECTED) {
    Serial.print(".");
    delay(100);
  }

  Serial.println();
  Serial.print("WiFi connected: ");
  Serial.println(WiFi.localIP());
}
```


### upload.sh
```bash
#!/bin/bash

set -eu

SERIAL_PORT="$1"


# ESP32-DevKitC
FQBN="esp32:esp32:esp32"

# Arduino UNO
# FQBN="arduino:avr:uno"


SKETCH="$(basename $PWD).ino"
ARGS="${@:2}"

arduino-cli upload \
    -b "$FQBN" \
    -p "$SERIAL_PORT" \
    "$ARGS"
```

```bash
./upload.sh /dev/ttyACM0
```


### serialmon.sh
```bash
#!/bin/bash

set -eu

SERIAL_PORT="$1"
BAUDRATE=115200

screen "$SERIAL_PORT" "$BAUDRATE"
```

```bash
./serialmon.sh /dev/ttyACM0
```

閉じるには`Ctrl+A k`を押した後に`y`を押して`Enter`。

### update.sh
まとめて実行する

```bash
#!/bin/bash

set -eu

SERIAL_PORT="$1"

./compile.sh && ./upload.sh "$SERIAL_PORT" && ./serialmon.sh "$SERIAL_PORT"
```

```bash
./update.sh
```


### compile.shの中身
同ディレクトリ中にある`.env`ファイルを読み込み、
C言語の定数として取り込まれる`build.defines`に手動で変数を列挙している
（nginx Dockerのtemplate置換のように定義済みの環境変数を自動で入れ込む改良もしたい）。

またDEFINESの中身をエスケープしたい（変数内に` -D`を許容したい）が、
まだやり方がわかっていない。


## 関連
- [Arduinoスケッチに安全に秘匿値を埋め込む｜o3c9｜note](https://note.com/o3c9/n/ne14c1817be11)
    - Arduino IDEのコマンドライン機能を使ったりMakefile作ったりしている
