---
title: "M5Stack用のピコピコ音源モジュール、五音ピコを作る"
emoji: "✨"
type: "tech"
topics:
  - "cpp"
  - "arduino"
  - "m5stack"
published: true
published_at: "2025-08-26 23:14"
---

みんな、こんにちは！みんなはゲームボーイみたいなピコピコ音源は好き？ボクは大好き！M5StackにはUNIT SYNTHというMIDIシンセサイザーユニットがあって色々な楽器の音は出せるんだけど、ピコピコ音は出せないんだよね。あと、ユニットの形だからM5Stack coreシリーズにスタックして使うこともできないよね。

そこで、今回はM5Stack coreシリーズにスタック可能な自作のピコピコ音源モジュールを作ることにしたよ。ピコピコ音源モジュールはXIAO RP2040でMozziというArduino用の波形合成ライブラリを動かして、RP2040からI2S出力でMAX98357AというI2Sアンプから音を出すようにしたよ。

https://youtu.be/q3di9m9mYm0

この記事は、M5Stack Global Innovation Contest 2025に応募するために五音ピコという作品でHacksterに登録した内容を日本向けに書き直したものだよ。

https://www.hackster.io/nananauno/go-on-pico-8-bit-style-synthesis-module-ea9673

## この記事はこんな人にオススメ
- M5Stackでお手軽にピコピコ音を出したい
- ピコピコ音大好き

## 五音ピコって？
五音ピコ (Go-On Pico)はM5Stack core2用のピコピコ音合成モジュールで、M5Stackを使用して誰でも簡単にレトロゲーム風の音を出せるようにすることと、最終的にはフォルマント合成というテクニックを使用してなんちゃって歌声合成ができることを目指しているよ。

![](https://storage.googleapis.com/zenn-user-upload/f65e8753f079-20250825.jpg =420x)
*五音ピコ（プロトタイプ）の外観*

五音ピコの名前は、M5Stackの5とモジュールが5つの音（矩形波×2ch、三角波×1ch、ノイズ×1ch、波形メモリ×1ch）を出せるように設計されているから五音（ごおん）って名前にしたよ。ピコはピコピコ音源だから。

## 特徴
五音ピコの特徴だよ！
- 8bit風ピコピコ音（矩形波×2ch、三角波×1ch、ノイズ×1ch、波形メモリ×1ch）*1
- シリアル経由の簡単制御
- M5Stack core2用のモジュールとしてスタック可能

*1: 波形メモリは未実装。

## 必要なもの
五音ピコを作成するために必要なものは以下の通りだよ。
- Seeed XIAO RP2040
- M5Stack BUS module
- MAX98357A Adafruitのブレークアウト基板（コピー品の方が安いよ）
- スピーカー 8Ω 1W

スピーカーはボクはM5StackのSYNTH UNITから取り外したものを使ったよ。薄型でモジュールに収まるスピーカーならOKだよ。

上記の材料に加えて、モジュールを制御するためのM5Stack core2が必要だよ。

## 環境
- Arduino IDE
- PlatformIO
- Mozzi

**Mozzi**はArduinoで動く波形合成ライブラリだよ。今回はXIAO RP2040でMozziを使ってピコピコ音を生成して出力しているよ。Mozziを使うことでピコピコ音に必要な矩形波や三角波を簡単に合成したりフィルターをかけたりすることができるよ。

## 五音ピコの作り方
ここでは五音ピコのモジュールの作り方を説明しているよ。今回作るのはプロトタイプで、M5StackのBUS module上に部品を実装していくよ。

![](https://storage.googleapis.com/zenn-user-upload/5371f341b6fb-20250826.png =420x)
*五音ピコのブロック図*

各部品同士の配線方法はブロック図に書いてあるからそれを見てね。

1. XIAO RP2040にピンヘッダを取り付けて、BUS module上に実装するよ。

2. MAX98357Aのブレークアウト基板（I2Sアンプ）にピンヘッダを取り付けて、BUS module上に実装するよ。スピーカー用の端子は取り外したいなら取り付けて、取り外し不要なら直接はんだ付けしてもいいよ。

3. XIAO RP2040のGPIO, 3.3V, GNDをI2Sアンプに接続するよ。リード線をはんだ付けしてね。

4. スピーカーをI2Sアンプに接続するよ。

5. BUS moduleのM-BUSから13番と14番のピンをXIAO RP2040へ接続するよ。これはM5Stackとのシリアル通信で使用されるよ。

6. M-BUSの5VとGNDをXIAO RP2040に接続するよ。これはXIAO RP2040に電源を供給するためだよ。

7. 全ての配線に間違いやはんだ不良が無いかどうかを確認するよ。

8. XIAO RP2040とPCをUSBケーブルで接続するよ。

9. Go-on-PicoをGithubからクローンしてArduinoIDEでビルドしてね。その後、XIAO RP2040をブートモードに入れて、ファームを書き込んでね。

完了！

https://github.com/nananauno/Go-on-Pico

## 五音ピコの使い方
M5Stack Core2から五音ピコを制御してピコピコ音を出力するのはとっても簡単！
Serial.beginで13, 14番のピンを開いて、シリアルにコマンドを書き込むだけだよ。

**制御コマンド**
コマンドのフォーマットは以下の通りだよ：
```
command,ch,note,velocity
```
**音を出す:** 1,0,60,127
指定のチャンネルでMIDIのノート番号60とベロシティ127で音を出すコマンドの例。

**音を止める:** 0,0,60,127
指定のチャンネルで音を止めるコマンドの例。

**音の種類**
ピコピコ音の種類はチャンネルによって決まっているよ。
CH0: 矩形波
CH1: 矩形波
CH2: 三角波
CH3: ノイズ

今のところ矩形波はDuty50%しか選べないけど、五音ピコモジュールの方ではDuty12.5%, Duty25%, Duty75%も出力できるように準備しているよ。

M5Stack Core2から五音ピコで音を鳴らす簡単な例を載せておくよ。参考にしてみてね。
```cpp
#include <M5Unified.h>

void sendSerialMessage(uint8_t command, uint8_t channel, uint8_t tone, uint8_t velocity) {
    char message[20];
    sprintf(message, "%02d,%02d,%03d,%03d\n", command, channel, tone, velocity);
    Serial2.print(message);
    Serial.print(message);
}

void setup() {
    auto cfg = M5.config();
    M5.begin(cfg);
    Serial.begin(115200);
    Serial2.begin(115200, SERIAL_8N1, 13, 14);

    M5.Display.setTextSize(2);
    M5.Display.setCursor(0, 40);
    M5.Display.println("BtnA: C4");
    M5.Display.println("BtnB: D4");
    M5.Display.println("BtnC: E4");
}

void loop() {
    M5.update();

    // BtnA → C4
    if (M5.BtnA.wasPressed()) {
        sendSerialMessage(1, 0, 60, 67); // note_on C4
        Serial.println("BtnA was pressed");
    }
    if (M5.BtnA.wasReleased()) {
        sendSerialMessage(0, 0, 60, 0);   // note_off C4
        Serial.println("BtnA was released");
    }

    // BtnB → D4
    if (M5.BtnB.wasPressed()) {
        sendSerialMessage(1, 0, 62, 67); // note_on D4
        Serial.println("BtnB was pressed");
    }
    if (M5.BtnB.wasReleased()) {
        sendSerialMessage(0, 0, 62, 0);   // note_off D4
        Serial.println("BtnB was released");
    }

    // BtnC → E4
    if (M5.BtnC.wasPressed()) {
        sendSerialMessage(1, 0, 64, 67); // note_on E4
        Serial.println("BtnC was pressed");
    }
    if (M5.BtnC.wasReleased()) {
        sendSerialMessage(0, 0, 64, 0);   // note_off E4
        Serial.println("BtnC was released");
    }
}
```

## 今後やりたいこと
- **歌声合成:** フォルマント合成を調整してお手軽に歌声合成ができるようにしたいな。
- **エンベロープ:** ゲームボーイの音源はエンベロープと言って音の鳴り始め〜音が鳴り終わるまでの変化（attack, decay, sustain, release）を制御できるようになっているよ。このエンベロープ機能も追加する予定だよ。
- **波形メモリ:** 任意のサンプリング音も鳴らせるように波形メモリ機能も追加予定だよ。

## どうして五音ピコを作ったの？
マイコンや周辺機能が高性能になったり、PCやクラウドで処理したりすることでお手軽に流暢な音声や歌声合成ができるようになったよね。でもマイコンって限られたリソースの中で完璧じゃなくてもいい、限られたリソースで工夫してなんとなくそれっぽいものができたら楽しいと思わない？ボクはそういうのが好き。

五音ピコはAquesTalk Picoっていう音声合成チップからインスピレーションを得ているよ。[AquestTalk Pico](https://www.a-quest.com/products/aquestalkpicolsi.html)ってArduino UNOと同じATMega328っていうマイコンで実現されていて、性能は全然高くないマイコンなんだけど、結構流暢に喋れてすごいなーって思ったよ。あとはYouTubeでゲームボーイの音源を使って「歌わせる」動画も色々あって、そういう動画からも刺激をもらったよ。限られた音だけで歌わせるってのが面白いよね。

## まとめ
M5Stack Core2にスタックしてピコピコ音をお手軽に鳴らせる五音ピコっていう自作モジュールの作り方と使い方を説明したよ。今は最初のプロトタイプ段階で機能もまだまだ実装中だけど、必要最低限でピコピコ音を楽しむことはできるようになっているよ。最終的な目標はフォルマントっていうテクニックを使ってなんちゃって歌声合成をすることだから、継続して機能を追加してきたいな！

みんなもM5Stackでお手軽にピコピコ音を楽しんでみてね！じゃあ、またね！

## 参考文献
https://qiita.com/rild/items/339c5c36f4c1ad8d4325