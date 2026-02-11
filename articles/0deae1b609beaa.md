---
title: "How to: M5Stackで超簡単にBluetoothスピーカーを作る"
emoji: "🔊"
type: "tech"
topics:
  - "arduino"
  - "m5stack"
  - "bluetooth"
  - "a2dp"
published: true
published_at: "2023-08-17 10:51"
---

## この記事でやること
周辺機能が豊富なM5Stack core2、買ってすぐに色々試すことができるのが特徴だね。このM5Stack core2を使ってBluetoothスピーカーを作ってみるよ。すごく簡単にできるから、みんなも試してみてね。コードの短さにびっくりすると思うよ。

https://twitter.com/nananauno/status/1691457019356811264

## 必要なもの
M5Stack core2でBluetoothスピーカーを作成するために必要なものは以下の通りだよ：
- M5Stack core2 ×1
- Arduino IDE
- 動作確認用のスマホなど

## ざっくり技術解説
Bluetoothスピーカーを作成するには、

1. スマホみたいなBluetooth機器からどうやって音楽を伝送しているの？
2. Bluetoothで伝送された音声データをどうやってスピーカーから出力するの？

を知る必要があるよね。ここでは、M5Stack core2でBluetoothスピーカーを作るときに最低限必要な２つの技術について説明するよ。

### Bluetooth A2DPプロファイル
１つ目のBluetooth機器から音楽を伝送する方法についてざっくりと説明するよ。

規格の細い内容はここでは省略するけど、Bluetoothでは色々な機器を接続できるようになっていて、機器ごとにプロファイルというものが定義されているよ。機器によって伝送するデータは多種多様なので、機器ごとにどのようなデータをやり取りするのかを決めているのがプロファイルだよ。例えば、みんなが一番良く使っているのは、Bluetoothキーボードやマウスじゃないかな？キーボードやマウスのような入力機器には、HIDプロファイルが使われているよ。

音楽を伝送するためのプロファイルも定義されていて、それがAdvanced Audio Distribution Profile (A2DP) だよ。これは、2chのデジタルオーディオを伝送するためのプロファイルだよ。

M5Stack core2に使われているESP32というマイコンにはBluetoothモジュールが搭載されているから、追加のモジュール無しでBluetooth A2DPに対応することができるよ。また、ESP32でA2DPを扱うためのライブラリも公開されているから、今回はこのライブラリを使っていくよ。

### I2Sパワーアンプ
２つ目のBluetoothで伝送された音声データをM5Stack core2のスピーカーから出力する方法についてざっくりと説明するよ。M5Stack core2のスピーカーがマイコンとどうやって接続されているかは回路図を見てね。

![](https://storage.googleapis.com/zenn-user-upload/da8b2f5a5f79-20230817.png)
[M5Stack, M5Stack core2 schematic](https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/docs/schematic/Core/CORE2_V1.0_SCH.pdf)

まずはスピーカー。回路図を見ると、M5Stack core2のモノラルスピーカーはNS4168というI2Sパワーアンプに接続されていることが分かるよ。パワーアンプは、マイコンから伝送された音声信号をスピーカーから出力するために信号を増幅するICだよ。I2SというのはInter-IC Soundの略で、IC間で音声データをデジタル伝送するためのシリアル通信規格だよ。I2Sは、音声データを左チャンネルと右チャンネルに分けて伝送し、クロック信号で同期するようになっているよ。

次にマイコン。回路図を見ると、マイコン側のGPIO0, GPIO12, GPIO2がそれぞれパワーアンプICのLRCK, BCLK, SADTAに接続されていることが分かるよ。LRCK, BCLK, SADTAはI2Sで使用するピンで、それぞれ以下のような役割があるよ。

- LRCK: SADTAの音声信号の左チャネル、右チャネルを区別するための信号
- BCLK: 音声信号を同期するための信号
- SADTA: 音声信号

以上が、Bluetoothスピーカーを作るために必要な技術の説明だよ。

## ライブラリのインストール
M5Stack core2で使用されているESP32でBluetooth A2DPを簡単に扱うためのライブラリが公開されているよ。今回はこのライブラリを使うので、Arduino IDEにライブラリをインストールするよ。

https://github.com/pschatzmann/ESP32-A2DP/

ESP32-A2DPはArduino IDEのLIBRARY MANAGERで検索しても出てこないので、Githubからzipを取得して、手動でインストールする必要があるよ。手順は以下の通りだよ。

1. GithubのESP-A2DPにアクセス
2. Code→Download ZIPを選択
3. Arduino IDEのSketchメニューからInclude Library→Add .zip library...と選択
4. Githubからダウンロードしたzipファイルを選択

次に、実際にコードを書いていくよ。

## サンプルコード

M5Stack core2をBluetoothスピーカーにするための全コードは以下の通りだよ。とても短くてびっくりだよね。これだけで、M5Stack core2をBluetoothスピーカーにすることができるよ。まずはコードをコピペしてスケッチをM5Stack core2に書き込んでみてね。

```cpp
#include <M5Core2.h>
#include <driver/i2s.h>
#include <BluetoothA2DPSink.h>

// Define I2S pins for audio output
#define I2S_BCK_PIN  12  // Bit Clock (BCK)
#define I2S_WS_PIN   0  // Word Select (WS)
#define I2S_DOUT_PIN 2  // Data Output (DOUT)

BluetoothA2DPSink a2dpSink;

void setup() {
  M5.begin();
  M5.Lcd.println("Bluetooth A2DP Speaker");
  M5.Axp.SetSpkEnable(true);

  // Initialize I2S audio output
  i2s_pin_config_t pinConfig = {
    .bck_io_num = I2S_BCK_PIN,
    .ws_io_num = I2S_WS_PIN,
    .data_out_num = I2S_DOUT_PIN,
    .data_in_num = I2S_PIN_NO_CHANGE
  };
  
  // Set up Bluetooth A2DP sink
  a2dpSink.set_pin_config(pinConfig);
  a2dpSink.start("M5Stack Speaker");
}

void loop() {
  // Main loop, nothing specific needed here
}
```

## サンプルコード解説
サンプルコードについて少し細かく説明するよ。

### ヘッダーファイル

```cpp
#include <M5Core2.h>
#include <driver/i2s.h>
#include <BluetoothA2DPSink.h>
```

#### M5Core2.h
M5Stack core2を使うために必要なヘッダーファイルだね。

#### driver/i2s.h
I2Sで音声信号を出力したり入力したりするときに必要なライブラリだよ。サンプルコードで直接何か関数を呼び出してはいないけど、I2Sのピン設定用のi2s_pin_config_t構造体を使用しているよ。

#### BluetoothA2DPSink.h
GithubのESP32-A2DPから取得したBluetooth A2DPを簡単に扱うためのライブラリだよ。Sinkというのは信号を受ける側の機器という意味だよ。Sourceというのもあって、これは信号を送信する側の機器だよ。今回は音声信号を受ける側なのでM5Stack core2はSinkになるよ。

ESP32-A2DPライブラリはSourceも対応しているから、音声信号を送信する側にもなることができるよ。

### ピンの設定

```cpp
// Define I2S pins for audio output
#define I2S_BCK_PIN  12  // Bit Clock (BCK)
#define I2S_WS_PIN   0  // Word Select (WS)
#define I2S_DOUT_PIN 2  // Data Output (DOUT)
```

マイコンとI2Sパワーアンプを接続しているピンを定義しているよ。M5Stack core2で使用されているNS4168に接続されているピンの番号はI2Sパワーアンプの項目で説明した通りだよ。

M5Stackの場合、本体の裏面にピン割り当てのラベルが貼り付けられているから、これを見ることでピン割り当てを知ることもできるよ。とっても便利だね！

### グローバル変数

```cpp
BluetoothA2DPSink a2dpSink;
```

ESP32-A2DPを使うためにBluetoothA2DPSinkクラスのインスタンス変数を作成しているよ。

### setup()
setup関数の内容について説明するよ。

#### I2Sパワーアンプのミュート解除

```cpp
M5.Axp.SetSpkEnable(true);
```

I2Sパワーアンプで説明した回路図をもう一回見て欲しいんだけど、I2SパワーアンプにCTRLというピンがあって、SPK_ENに接続されているのが分かるよね。このCTRLピンはパワーアンプの出力をMute/Unmuteするためのピンになっているよ。M5Stack core2がリセットされた直後はアンプがミュート状態になっているよ。ということは、スピーカーから音声を出力するためには、このCRTLを使ってアンプのミュートを解除しておく必要があるよ。

M5Stack core2の回路図を見ると、SPK_ENはAXP192という電源管理ICに接続されていることが分かるよ。これを制御するための関数がM5.Axp.SetSpkEnableで、trueを渡すことでパワーアンプのミュートが解除されるよ。

![](https://storage.googleapis.com/zenn-user-upload/4258f58d16e7-20230817.png)
[M5Stack, M5Stack core2 schematic](https://m5stack.oss-cn-shenzhen.aliyuncs.com/resource/docs/schematic/Core/CORE2_V1.0_SCH.pdf)

#### I2Sのピン設定

```cpp
// Initialize I2S audio output
  i2s_pin_config_t pinConfig = {
    .bck_io_num = I2S_BCK_PIN,
    .ws_io_num = I2S_WS_PIN,
    .data_out_num = I2S_DOUT_PIN,
    .data_in_num = I2S_PIN_NO_CHANGE
  };
```

i2s_pin_config_t構造体を使用してM5Stack core2のI2Sパワーアンプに接続されているピンを設定するよ。ピンは番号はさっきのピン設定で定義した通りだよ。名前は見たら分かるよね。.data_in_numはマイクから音声入力するときに使用するピンで、今回は使用しないのでI2S_PIN_NO_CHANGEを指定しているよ。

```cpp
  // Set up Bluetooth A2DP sink
  a2dpSink.set_pin_config(pinConfig);
```

a2dpSinkインスタンス変数のset_pin_configに先程設定したピンの設定を渡すことで、ESP32-A2DPライブラリに対して、音声を出力するためのI2Sピンを設定することができるよ。これでスピーカーへ出力するための設定は完了だよ。

#### A2DPを有効にする

```cpp
a2dpSink.start("M5Stack Speaker");
```

a2dpSinkインスタンス変数のstartを呼び出すことで、A2DPを使用できる状態になるよ。ここでstartに渡している文字列はBluetooth機器で表示される名前だよ。

### loop()
loop関数は特に処理は必要ないよ。

これだけで、Bluetooth A2DPが使用可能な状態になったよ。サンプルスケッチをM5Stack core2に書き込んで動作確認をしてみるよ。

## 動作確認

スマホなどの手元のBluetooth対応機器で動作確認をするよ。M5Stack core２にサンプルコードのスケッチを書き込めたら、スマホのBluetoothの設定を開いてね。機器一覧に"M5Stack Speaker"が表示されているはずだよ。M5Stack Speakerに接続してスマホで音楽を再生してみてね。

## まとめ
最後まで読んでくれてありがとう！周辺機能が豊富なM5Stack core2を使うことで、とても簡単にBluetoothスピーカーが実現できることが分かったよね。Bluetoothスピーカーで使われている技術は以下の２つだったね。

- Bluetooth A2DP
- I2S

ここで説明した内容は本当に基本的なところだけで、Bluetoothスピーカーとしては足りていない機能もあるんじゃないかな？例えば、Bluetoothスピーカー側に再生、停止、音量ボタンとかがあった方が便利だよね。ここからは自分なりに色々発展させたオリジナルのBluetoothスピーカーを作ってみてね。じゃあ、またね！