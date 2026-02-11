---
title: "How to: M5Stackで超簡単にBluetoothキーボードを作る"
emoji: "⌨️"
type: "tech"
topics:
  - "arduino"
  - "m5stack"
  - "keyboard"
  - "esp32"
published: true
published_at: "2025-02-09 10:49"
---

みんな、こんにちは！M5Stackを使ってワイヤレスでPCを操作したいって思ったことない？ボクは超お手軽な無線キーボードが作りたくて、M5Stack AtomLiteをBluetoothキーボードとして使う方法を試してみたよ！

AtomLiteはコンパクトなデバイスだけど、ESP32を搭載しているからBluetoothを使用してキーボードのようなHID (Human Interface Device) として動作させることができるよ。今回は、このAtomLiteを使って、**ボタンを押すとPCに「Hello」と入力するBluetoothキーボード**を作る方法を紹介するよ！

https://x.com/nananauno/status/1888385635469459664

## この記事でやること
M5Stackシリーズの**M5Stack AtomLite**を使って、超簡単なBluetoothキーボードを作るよ！ 
ボタンを押すとPCに「Hello」と入力するようにするよ。あと、キーボードとは関係ないけど、電源を入れても何も反応しないと寂しいから、ボタンのRGB LED (NeoPixel) を緑色に点灯させるよ。

AtomLiteは有線のUSBキーボードにはなれないんだけど、一方でBluetoothならとてもお手軽にキーボード作れそうだったから今回はそれを実現する方法を解説するよ！

## 必要なもの
- M5Stack **AtomLite**
- **開発環境**（Arduino IDE or PlatformIO）
- **USB-Cケーブル**（PCと接続するため）

**ライブラリ：**
- ESP32-BLE-Keyboard
- FastLED
- M5Unified

### M5Stack AtomLiteって？
M5Stack AtomLiteは、M5Stackさんが作ったESP32を搭載した超小型の開発ボードだよ！
- **サイズがわずか24×24mm**のコンパクトボディ
- **Wi-Fi & Bluetooth**モジュール搭載でワイヤレス通信が可能
- **ボタン1つ、RGB LED 1つ、GPIO拡張可能**
- **低消費電力**でバッテリー駆動もOK

小さいけどパワフルだから、IoTデバイスやBluetoothデバイスの開発にピッタリだよ！

https://ssci.to/6262

## ESP32用のBLEキーボードライブラリ
AtomLiteに使用されているESP32にはBluetoothモジュールが標準搭載。ESP32用のBLEキーボードのライブラリ`ESP32-BLE-Keyboard`というのがあって、このライブラリを使用することで、超簡単に**Bluetoothキーボードを作ることができる**よ。

AtomLiteに搭載されているESP32は**Bluetooth4.2 BR/EDR+BLEに対応**していて、BLE（Bluetooth Low Energy）を利用できるよ。一方でStampS3などのESP32-S3を搭載した開発ボードは、**Bluetooth 5.0（LE）に対応**していて、サポートしている規格が異なるから使える機能にも差がある点には注意が必要だね。

## ライブラリのインストール
まずは、必要なライブラリをインストールするよ。

**Arduino IDEの場合**
1. GithubからESP32-BLE-Keyboardのzipをダウンロード
2. Arduino IDEで"Sketch" -> "Include Library" -> "Add .ZIP Library..."と選択
3. `FastLED` をArduino IDEのライブラリから検索してインストール
4. 同じく`M5Unified`をインストール

https://github.com/T-vK/ESP32-BLE-Keyboard

**PlatformIOの場合**
`platformio.ini` では以下のようにlib_depsを記述しているよ：

```ini
[env:m5stack-atom]
platform = espressif32
board = m5stack-atom
framework = arduino
board_build.partitions = huge_app.csv
lib_deps =
  t-vk/ESP32 BLE Keyboard @ ^0.3.2
  m5stack/M5Unified @ ^0.2.3
  fastled/FastLED @ ^3.9.13
```

### パーティション設定
👆のplatformio.iniでは、パーティション設定を `huge_app.csv` にしているよ。理由は、デフォルトのパーティション設定ではアプリ領域が少なく、BLEのライブラリを使用すると残りのサイズがほとんどなくなってしまうためだよ。`huge_app.csv` を使うことで、アプリケーションが使える領域を増やし、より安定した動作を確保できるよ！

Arduino IDEでは、"Tools" -> "Partition Scheme:" -> "Huge APP"を選んでね。

## サンプルコード
```cpp
#include <M5Unified.h>
#include <BleKeyboard.h>
#include <FastLED.h>

#define LED_PIN 27       // NeoPixelの接続ピン
#define NUM_LEDS 1       // LEDの個数
CRGB leds[NUM_LEDS];     // LED配列

BleKeyboard bleKeyboard("AtomLite Keyboard");  // Bluetoothデバイス名

void setup() {
    M5.begin();
    bleKeyboard.begin();          // Bluetoothキーボードを開始
    // FastLEDのセットアップ
    FastLED.addLeds<WS2812, LED_PIN, GRB>(leds, NUM_LEDS);
    leds[0] = CRGB::Green;  // LEDを緑色に設定
    FastLED.show();         // LEDの状態を更新
}

void loop() {
    M5.update();

    if (M5.BtnA.wasPressed()) { // ボタンが押されたら
        if (bleKeyboard.isConnected()) {  // Bluetooth接続されているか確認
            bleKeyboard.print("Hello");   // "Hello"をPCに入力
        }
    }
}
```

## サンプルコード解説
1. `BleKeyboard` を使ってAtomLiteをBluetoothキーボードとして設定しているよ
2. ボタン（M5.BtnA）を押すと「Hello」を送信するよ
3. NeoPixel（G27に接続）を緑色に点灯しているよ

## 動作確認
👆のコードをArduino IDEやPlatformIOでビルドして、AtomLiteに書き込んでみてね。以下のように動作すれば成功だよ！

1. AtomLiteの電源を入れると「AtomLite Keyboard」というBluetoothデバイスが表示される
2. PCでペアリング
3. テキスト入力できるアプリ（メモ帳など）を開く
4. ボタンを押すと「Hello」が入力される！

## まとめ
AtomLiteを使ったBluetoothキーボードはうまく動いたかな？
今回はM5Stack AtomLiteを使って、超簡単にBluetoothキーボードを作る方法をまとめてみたよ。

ポイントは、
- AtomLiteで使用されているESP32にはBluetoothが標準搭載されている
- `ESP32-BLE-Keyboard` を使うと、お手軽にESP32をBluetooth HIDデバイスにできる

AtomLiteの裏には6つのGPIOがあるから、ここにスイッチを接続すればもっと多くのキーを備えたBluetoothキーボードを作れそうだね。基本的なコードはこの記事で紹介した通りだから、ここからさらに色々な機能を追加していって自分だけのキーボードを作ってみてね！

**この記事のほとんどはChatGPT (AI NANANA M5) によって生成されたよ**