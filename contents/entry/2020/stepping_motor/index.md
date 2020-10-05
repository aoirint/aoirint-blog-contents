---
canonical_url: ./
title: ロータリエンコーダ付きステッピングモータをArduinoで制御して角度を取得する
# og_image:
# twitter_card: summary_large_image
og_description: ロータリエンコーダ付きステッピングモータをArduinoで制御して角度を取得する
date: '2020-10-06 07:00:00'
draft: false
category: 記事
tags:
  - ステッピングモータ
  - ロータリエンコーダ
  - Arduino
---
# ロータリエンコーダ付きステッピングモータをArduinoで制御して角度を取得する

## 機材
- Arduino UNO（中華）
- ロータリエンコーダ付きステッピングモータ
    - PKP214U06A-R2EL
    - [PKP214U06A-R2EL-L｜PKPシリーズ／PKシリーズ｜ステッピングモーター｜オリエンタルモーター株式会社](https://www.orientalmotor.co.jp/products/detail.action?hinmei=PKP214U06A-R2EL-L)
      - モータ部説明書（PDF、HM-7433J.pdf）
      - ロータリエンコーダ部説明書（PDF、HM-7439JE.pdf）
      - [特性図（画像）](https://www.orientalmotor.co.jp/file_addon/products/st/image/tj_pkp214u06a-r2el-l_d.gif)（DC24V駆動時）
      - モータ部
        - 2相
        - ユニポーラ5本リード線
        - 基本ステップ角度1.8°
        - AC 50/60Hz 0.5kV 絶縁耐圧（1分間）
      - ロータリエンコーダ部
        - 分解能 200 パルス/回転（pulse/revolution）
        - A相、B相、Z相：3チャンネル出力
        - DC5V駆動
- モータドライバ
    - SLA7078MPRT
    - [SLA7078MPRT ｜サンケン電気](https://www.semicon.sanken-ele.co.jp/ctrl/product/category/2Ph_StepMotorUnp/detail/?product=SLA7078MPRT)
    - [２相ステッピングモータードライバー　ユニポーラ駆動用　ＳＬＡ７０７８ＭＰＲＴ: 半導体 秋月電子通商-電子部品・ネット通販](https://akizukidenshi.com/catalog/g/gI-08015/)
    - [データシート（PDF、sla7073mprt_ds_jp.pdf）](https://www.semicon.sanken-ele.co.jp/sk_content/sla7073mprt_ds_jp.pdf)
    - μステップ対応品
    - 実使用電圧 10-44V
- モータドライバ基板
    - SEC20120330A
    - [エレ・メカ・ホビーショップＳＥＣ 資料置き場](http://sec-suzuki.com/newpage333.htm)
        - [新型 ２相ステップドライバ資料（PDF、step-2p-v1.pdf）](http://sec-suzuki.com/step-2p-v1.pdf)

## ステッピングモータの制御
### 配線
ステッピングモータ本体からは、左から黒・緑・橙・青・赤の5本のケーブルが出ている。
これは説明書を見ると
黒・緑がA相（A・<span style="text-decoration: overline;">A</span>）、
青・赤がB相（B・<span style="text-decoration: overline;">B</span>）、
橙が電源になっている。
モータドライバ基板の対応する端子にこれらを接続する。

モータ電源として24V DC電源（[ATS065-P240](https://akizukidenshi.com/catalog/g/gM-06962/)）を使用した。

### ドライバ設定
μステップ機能（角度をより細かく制御できる）を使い、
励磁方式をW1-2相励磁（4分割）にするため、ドライバのM3端子をHIGHにする。
今回使ったドライバ基板ではDIPスイッチの4番をONにする。

- [第3回 ドライブICの制御方式「励磁方式」 | 特集 | NPM 日本パルスモーター株式会社](https://www.pulsemotor.com/feature/steppingmotordrive3.html)

ドライバ基板上の半固定抵抗（Refに接続）を使って、
カレントダウン（過熱防止のための電流カット機能）時の電流と
通常時の電流をできるだけ絞る（反時計回りで絞れる）。
実際に回すときは、求められるトルクと発熱のトレードオフで調節すると思われる。

### 制御コード
以下のような機能を実現する簡単な制御コードを書いた。要[TimerOne](https://github.com/PaulStoffregen/TimerOne)。

- パルスの開始・停止
- パルス幅の切り替え
- カレントダウンの切り替え
- 回転方向の切り替え

`SteppingMotor.ino`として新規タブに貼り付ける。

`RPM_PER_PULSE_KHZ`は特性図を見ると、パルス速度 1 kHzのとき回転速度 300 r/minとあるので300を使う。

```c
// ステッピングモータ 制御コード

#include <TimerOne.h>

// ピンはどこでもOK
#define PIN_MTR_DIR 5
#define PIN_MTR_CK 6
#define PIN_MTR_CD 7

// 特性図
// https://www.orientalmotor.co.jp/file_addon/products/st/image/tj_pkp214u06[].gif
#define RPM_PER_PULSE_KHZ 300

bool pulseIsHigh = false;

void initSteppingMotor() {
  pinMode(PIN_MTR_DIR, OUTPUT);
  pinMode(PIN_MTR_CK, OUTPUT);
  pinMode(PIN_MTR_CD, OUTPUT);

  digitalWrite(PIN_MTR_DIR, LOW);
  digitalWrite(PIN_MTR_CK, LOW);
  digitalWrite(PIN_MTR_CD, HIGH);

  Timer1.initialize(1000*1000);
}

void stopPulse() {
  Timer1.detachInterrupt();
}
void startPulse() {
  Timer1.attachInterrupt(switchPulse);
}
void switchPulse() {
  pulseIsHigh = !pulseIsHigh;
  digitalWrite(PIN_MTR_CK, pulseIsHigh ? HIGH : LOW);
}

void setPulseLengthMillis(float pulseMillis) {
  int pulseMicros = pulseMillis * 1000;
  if (pulseMicros < 1) pulseMicros = 1;
  Timer1.setPeriod(pulseMicros);
}
void setPulseLengthByKHz(float khz) {
  float pulseMillis = 1.0 / khz;
  setPulseLengthMillis(pulseMillis);
}
void setPulseLengthByRPM(float rpm) {
  float pulseKHz = rpm / RPM_PER_PULSE_KHZ;
  setPulseLengthByKHz(pulseKHz);
}

void disableCurrentDown() {
  digitalWrite(PIN_MTR_CD, LOW);
}
void enableCurrentDown() {
  digitalWrite(PIN_MTR_CD, HIGH);
}

void setCW() {
  digitalWrite(PIN_MTR_DIR, LOW);
}
void setCCW() {
  digitalWrite(PIN_MTR_DIR, HIGH);
}
```


### 実行コード
以下のようにテスト用の実行コードを作った。
30rpm、300rpm、1000rpm、300rpm、30rpm、停止、のように回転し、一巡するごとに回転方向を反転する（あまり急激に変化させると脱調してしまい回らない）。

```c
// ステッピングモータ 実行コード

int prevPhase = -1;
bool isClockwise = true;

void setup() {
  Serial.begin(115200);

  initSteppingMotor();
}

void loop() {
  long now = millis();
  int interval = 1000;
  int phase = (now / interval) % 6;


  if (phase != prevPhase) {
    Serial.print("[");
    Serial.print(now);
    Serial.print("] ");
    if (isClockwise) Serial.print("(C) ");
    else             Serial.print("(A) ");

    if (phase == 0) {
      Serial.println("stop");
      stopPulse();
      enableCurrentDown();
    }
    else if (phase == 1) {
      Serial.println("30rpm");

      disableCurrentDown();

      if (isClockwise) setCW();
      else             setCCW();
      isClockwise = !isClockwise;

      setPulseLengthByRPM(30);
      startPulse();
    }
    else if (phase == 2) {
      Serial.println("300rpm");
      setPulseLengthByRPM(300);
    }
    else if (phase == 3) {
      Serial.println("1000rpm");
      setPulseLengthByRPM(1000);
    }
    else if (phase == 4) {
      Serial.println("300rpm");
      setPulseLengthByRPM(300);
    }
    else if (phase == 5) {
      Serial.println("30rpm");
      setPulseLengthByRPM(30);
    }
    else {
      Serial.println("nope");
    }
  }

  prevPhase = phase;
}
```



## ロータリエンコーダによる角度取得

### 配線
ロータリエンコーダ本体からは、左から白・橙・黄・青・緑・茶・赤・黒の8本のケーブルが出ている。
これは説明書を見ると
赤・茶がA相（A+・A-）、
緑・青がB相（B+・B-）、
黃・橙がZ相（Z+、Z-）、
白がVCC（5V）、
黒がGNDになっている。
+-はそれぞれ出力が反転しているだけなので、
各相1本だけを接続すればいい。
Z+は軸が一周するタイミングで正になる端子で、誤差修正に使う。
よって、赤・緑・黃・白・黒の5本を接続する。

ロータリエンコーダの読み取りには割り込みを使うので、
A+、B+は2番、3番ピンに接続する。
Z+は4番ピン（これはどこでもOK）に接続することにする。
VCCとGNDはArduinoの5V、GNDに接続すればいい。

### 計算コード

- [ロータリーエンコーダを使う part 1 : 外部割込みとチャタリング対策 – jumbleat](https://jumbleat.com/2016/12/17/encoder_1/)

この記事を参考にして、ロータリエンコーダの角度計算をするコードを書いた。
A相B相のパターンに基づく計算はそのままで、不要な部分の除去とZ相によるリセット、オーバーフロー防止のためカウント超過時のリセットを追加した。
なおPPRは200なので、1.8°の分解能になる。

`RotaryEncoder.ino`として新規タブに貼り付ける。

```c
// ロータリエンコーダ 計算コード

#define PPR 200 // p/r, pulse per revolution

#define PIN_RTE_A 2
#define PIN_RTE_B 3
#define PIN_RTE_Z 4

#define NONE 0
#define CW 1
#define CCW 2

volatile char direction = NONE; // NONE, CW, CCW
volatile int steps = 0;

volatile bool prevPinA = false;
volatile bool prevPinB = false;
volatile bool prevPinZ = false;

void initRotaryEncoder() {
  pinMode(PIN_RTE_A, INPUT_PULLUP);
  pinMode(PIN_RTE_B, INPUT_PULLUP);
  pinMode(PIN_RTE_Z, INPUT_PULLUP);

  attachInterrupt(0, onRotaryEncoderPulse, CHANGE); // pin2
  attachInterrupt(1, onRotaryEncoderPulse, CHANGE); // pin3
}

int getRotaryEncoderStep() {
  return steps;
}
float getRotaryEncoderAngleRate() {
  return (float)steps / PPR;
}
float getRotaryEncoderAngleDegrees() {
  return getRotaryEncoderAngleRate() * 360.0f;
}
float getRotaryEncoderAngleRadians() {
  return getRotaryEncoderAngleRate() * 2 * PI;
}

void onRotaryEncoderPulse() {
  bool pinZ = digitalRead(PIN_RTE_Z);
  if (pinZ && prevPinZ) steps = 0;

  bool pinA = !digitalRead(PIN_RTE_A);
  bool pinB = !digitalRead(PIN_RTE_B);

  if (pinA != prevPinA || pinB != prevPinB) {
    if (direction == NONE) {
      if (pinA) direction = CW;
      if (pinB) direction = CCW;
    }
    else {
      if (!pinA && !pinB) {
        if      (direction == CW  && prevPinB) steps++;
        else if (direction == CCW && prevPinA) steps--;

        if (steps < 0)    steps += PPR;
        if (PPR <= steps) steps -= PPR;

        direction = NONE;
      }
    }
  }

  prevPinA = pinA;
  prevPinB = pinB;
  prevPinZ = pinZ;
}
```

### 実行コード
以下のようにテスト用の実行コードを作った。
感電に注意しつつ、手で軸を回して角度が変化することを確認する。

```c
// ロータリエンコーダ 実行コード

void setup() {
  Serial.begin(115200);

  initRotaryEncoder();
}

void loop() {
  Serial.println(getRotaryEncoderAngleDegrees());
  delay(10);
}
```


## ステッピングモータ+ロータリエンコーダ

### シリアル通信で送られてきたJSONをパースする
シリアル通信で改行文字で区切られたJSONをやり取りするため、`SerialJsonLineReader.ino`を作った。要[ArduinoJson](https://github.com/bblanchon/ArduinoJson)。

```c
// Requirements:
// #include <ArduinoJson.h>

String serialBuffer = "";

bool nextSerialLine(String *serialLine) {
  *serialLine = "";

  while (Serial.available() > 0) {
    int ch = Serial.read();
    if (ch != -1) {
      if (ch == '\n') {
        *serialLine = serialBuffer;
        serialBuffer = "";
        return true;
      }

      serialBuffer += (char) ch;
    }
  }

  return false;
}

bool nextSerialJson(JsonDocument *serialJson, bool *jsonError) {
  String serialLine = "";

  *jsonError = false;
  serialJson->clear();

  if (nextSerialLine(&serialLine)) {
    DeserializationError error;
    error = deserializeJson(*serialJson, serialLine);

    if (error == DeserializationError::Ok) {
      return true;
    }

    *jsonError = true;
    serialJson->clear();
  }

  return false;
}
```

### シリアル通信からステッピングモータ制御+角度取得

```json
{"rpm": 300}
```

このようなJSONをシリアル通信でArduinoに送ることで、
ステッピングモータが回転するようにし、
また逐次回転角度をシリアル通信で送る実行コードを作った。
要`SteppingMotor.ino`、
`RotaryEncoder.ino`、
`SerialJsonLineReader.ino`。

```c
#include <ArduinoJson.h>

#define MIN_RPM 30
#define MAX_RPM 3000
float rpm = 0;

void setup() {
  Serial.begin(115200);

  initRotaryEncoder();
  initSteppingMotor();
}

void loop() {
  float angleDegrees = getRotaryEncoderAngleDegrees();

  if (rpm < MIN_RPM) {
    rpm = 0;
    stopPulse();
    enableCurrentDown();
  }
  else {
    setPulseLengthByRPM(rpm);
    startPulse();
  }

  StaticJsonDocument<255> msg;
  msg["angle"] = angleDegrees;
  msg["rpm"] = rpm;

  serializeJson(msg, Serial); Serial.println();
}

void serialEvent() {
  StaticJsonDocument<255> msg;
  bool jsonError = false;

  if (nextSerialJson(&msg, &jsonError)) {
    float _rpm = msg["rpm"];
    if (_rpm < MIN_RPM) _rpm = 0; // current down
    if (MAX_RPM < _rpm) _rpm = MAX_RPM;

    rpm = _rpm;
  }
  else if (jsonError) {
    // StaticJsonDocument<255> response;
    // serializeJson(response, Serial); Serial.println();
  }
}
```
