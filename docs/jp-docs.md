# Balsun 国内向け情報

## Balsun
Balsun - Balloon adaptive link board with Wi-SUN は, 大気球実験向けに作ったWi-SUNチップを搭載したArduino UNO R4 Minima 互換ボードです.
大気球実験PIが, 地上系/搭載系で簡単に用いることができる汎用ボードとして開発されています.
なお, 開発者や開発者の所属機関が性能を保証する性質のものではありません.

## 今後
現在アルファ版作成の準備として製作業者の選定, 依頼を進めています.

## Getting started
Arduinoとして使う方法を説明します.
これらは, 試験用の仮の方法であり, 将来的に更新される予定です.

### ブートローダーの書き込み
[こちら](https://zenn.dev/ichirowo/articles/6aa1614e102bce) も参考になります.

まずは, USBの接続状態を確認できるツールをインストールします.
例えば, [USB Device Tree Viewer](https://www.uwe-sieber.de/usbtreeview_e.html) が利用できます.

次に, [Renesas Flash Programmer](https://www.renesas.com/jp/ja/software-tool/renesas-flash-programmer-programming-gui#overview) をダウンロード, インストールします.

最後に, [Arduino IDE](https://www.arduino.cc/en/software) をインストールして, Arduino UNO R4 Minimaのための設定をしておきます.


Balsun基板の, Boot pin (左下から2番目) とGNDをショートして, USB type-C経由でコンピューターと接続します.
その後, Resetボタンをおす (または, Reset pinとGNDのショート) と, デバイスが認識されます.
USB Device Tree Viewerに, RA USB Boot(CDC)(COMxx) などと表示されます.

Renesas Flash Programmerを起動して, ファイル→新しいプロジェクトを作成を選択し, マイクロコントローラにRA, COM portに確認したポートを指定します. その他は適当に入力してください.
または, "Connect Settings" タブからToolに COM port, Speedに115200 bps, Tool Details... に確認したCOMポートを指定します.

プログラムファイルの「参照」をクリックし、UNO R4 MINIMAのブートローダーを指定します。以下のフォルダから、「`dfu_minima.hex`」を指定します。
`C:\Users\<ユーザー名>\AppData\Local\Arduino15\packages\arduino\hardware\renesas_uno\1.0.1\bootloaders\SANTIAGO`
もしくは
`C:\Users\<ユーザー名>\AppData\Local\Arduino15\packages\arduino\hardware\renesas_uno\1.0.2\bootloaders\UNO_R4`

スタートボタンをおすと, 書き込みが始まります. 緑色で"操作が成功しました"と表示されたら完了です.

その後, Arduino IDEを開いて, 適当なスケッチを書き込めが, Arduinoとして認識されるはずです.

### Wi-SUNチップの利用

以下のスケッチをMaster, Slaveにそれぞれ書き込みます.

 - master
 ```
UART SerialX(23,24);

void setup() {

  pinMode(LED_BUILTIN, OUTPUT);
  Serial.begin(9600); // USB type-C
  SerialX.begin(115200, SERIAL_8N1); // Serial pin for Wi-SUN
    
  R_IOPORT_PinCfg(NULL, BSP_IO_PORT_02_PIN_04, IOPORT_CFG_PORT_DIRECTION_OUTPUT);

  R_IOPORT_PinWrite(NULL, BSP_IO_PORT_02_PIN_04, BSP_IO_LEVEL_HIGH);
  delay(100);
  R_IOPORT_PinWrite(NULL, BSP_IO_PORT_02_PIN_04, BSP_IO_LEVEL_LOW);
  delay(100);
  R_IOPORT_PinWrite(NULL, BSP_IO_PORT_02_PIN_04, BSP_IO_LEVEL_HIGH);
  delay(100);
  SerialX.println ("atstart BORDER");
  delay(100);
  SerialX.println("save");
  delay(100);
  SerialX.println("vernum");
  delay(10000);
  SerialX.println("rantsw 1"); // 1 for chip, 2 for SMA
  delay(100);

  digitalWrite(LED_BUILTIN, HIGH);
  //if (SerialX.available() > 0){
  //  String msg = SerialX.readString();
  //  Serial.print(msg);
  //} 
}

const int adc_pin = A5;
int counter = 0;

void loop() {
  if (Serial.available() > 0){
    String cmd = Serial.readString();
    SerialX.print(cmd);
  }
  if (SerialX.available() > 0){
    String msg = SerialX.readString();
    Serial.print(msg);
  } 

  counter++;

  delay(100);
}

 ```

 - slave
 ```
 UART SerialX(23, 24);

int split(String data, char delimiter, String* dst) {
  int index = 0;
  int arraySize = (sizeof(data)) / sizeof((data[0]));
  int datalength = data.length();

  for (int i = 0; i < datalength; i++) {
    char tmp = data.charAt(i);
    if (tmp == delimiter) {
      index++;
      if (index > (arraySize - 1)) return -1;
    } else dst[index] += tmp;
  }
  return (index + 1);
}

int pin2_counter = 0;
int pin3_counter = 0;
int pin4_counter = 0;

void setup() {

  pinMode(2, OUTPUT);
  pinMode(3, OUTPUT);
  pinMode(4, OUTPUT);

  Serial.begin(9600); // USB type-C

  SerialX.begin(115200, SERIAL_8N1); // Serial pin for Wi-SUN

  R_IOPORT_PinCfg(NULL, BSP_IO_PORT_02_PIN_04, IOPORT_CFG_PORT_DIRECTION_OUTPUT);

  R_IOPORT_PinWrite(NULL, BSP_IO_PORT_02_PIN_04, BSP_IO_LEVEL_HIGH);
  delay(100);
  R_IOPORT_PinWrite(NULL, BSP_IO_PORT_02_PIN_04, BSP_IO_LEVEL_LOW);
  delay(100);
  R_IOPORT_PinWrite(NULL, BSP_IO_PORT_02_PIN_04, BSP_IO_LEVEL_HIGH);
  delay(100);

  delay(10000);
  SerialX.println("rantsw 1"); // 1 for chip, 2 for SMA
}

int counter = 0;
const int adc_pin = A5;

void loop() {
  if (Serial.available() > 0){
    String cmd = Serial.readString();
    SerialX.print(cmd);
  }

  if (SerialX.available() > 0){
    String msg = SerialX.readString();
    Serial.print(msg);

    int idx = msg.indexOf("tcpr");

    if(idx == 1){
      String buf[10];
      split(msg, ' ', buf);
      int cmdid = buf[2].toInt();

      if(cmdid==1){
          Serial.print("debug command");
          SerialX.println("tcps 2001:db8::1 0000");
      } else if (cmdid==2){
        SerialX.println("tcps 2001:db8::1 0002");
        pin2_counter = 300;
        digitalWrite(2, HIGH);
      } else if (cmdid==3){
        SerialX.println("tcps 2001:db8::1 0003");
        pin3_counter = 300;
        digitalWrite(3, HIGH);
      } else if (cmdid==4){
        SerialX.println("tcps 2001:db8::1 0004");
        pin4_counter = 100;
        digitalWrite(4, HIGH);
      } else if (cmdid==9){
        pin2_counter = 0;
        pin3_counter = 0;
        pin4_counter = 0;
        digitalWrite(4, HIGH);
      } else if (cmdid==9999){
          digitalWrite(2, HIGH);
          digitalWrite(3, HIGH);
          digitalWrite(4, HIGH);
          delay(10000);
          digitalWrite(2, LOW);
          digitalWrite(3, LOW);
          digitalWrite(4, LOW);
      }
    }


  }
  if (pin2_counter>0){pin2_counter--;} else {digitalWrite(2, LOW);}
  if (pin3_counter>0){pin3_counter--;} else {digitalWrite(3, LOW);}
  if (pin4_counter>0){pin4_counter--;} else {digitalWrite(4, LOW);}

  if (counter % 1000 == 999){
    int adc = analogRead(adc_pin);
    char sendPacket[24] = "";
    sprintf(sendPacket, "tcps 2001:db8::1 %04d", adc);
    SerialX.println(sendPacket);
    counter = 0;
  }

  counter++;
  delay(100);
}

 ```

書き込んだら, slaveの電源を投入し, masterはusbでコンピューターに接続してシリアルモニタを開きます.
ボーレートは9600です. (Source codeをいじれば適当に変えられます.)

```
atstart BORDER
```
とすると, response `atstart 1(BORDER)` が帰ってくるはずです.
これで, 設定をRAMに書き込むために, `save` と書き込むと, `save parameter is saved` とResponseが帰ってくるはずなので,
`reset` を打ってWi-SUNのチップの再立ち上げをします.
以下の出力が見えるはずです.
```
18:44:34.164 -> reset delay 0sec
18:44:34.164 -> inf 01,01,0,0       {    WSN: system booted.                      }
18:44:34.164 ->
////////////////////////////////////////////////////////////
18:44:34.164 -> // Copyright (C) 2020 Nissin Systems Co.,Ltd.
18:44:34.164 -> // EW-WSN-FAN-1.0.56.60 ROHM BP35C5(ROHM ML7436N:ML7421)
18:44:34.164 -> // Wi-SUN Profile for FAN (Dec 24 2020 12:55:57)
18:44:34.164 -> ////////////////////////////////////////////////////////////
18:44:34.164 -> auto start 1 (BORDER)...
18:44:34.164 -> init 1(BORDER)
18:44:34.164 -> inf 2b,62,0,5       {   FMng: changed fan join state (0 -> 5)     }
```

しばらく待つと, Slave機器を認識して, addressを与えます.

```
18:44:34.164 -> >inf 40,2b,bb48,4f5  {   NBRS: added new neighbor <001d129f35c54096>}
nebr <001d129f35c54096>: 256/127(-47)	<::>
18:45:07.166 -> >
inf 40,2d,3f7f,bb48 {   NBRS: added address <2001:db8::a9> to <001d129f35c54096>}
18:45:22.104 -> >
inf 4c,39,9d8c,9d60 {    RPL: added node <2001:db8::a9> => <2001:db8::1>}

```
その後, SlaveはADCの値を読んで定期的にMasterと通信してくるので,
```
tcpr <2001:db8::a9> 0040
18:47:14.445 -> >
tcpr <2001:db8::a9> 0040
18:48:55.375 -> >
tcpr <2001:db8::a9> 0040
18:50:36.343 -> >
tcpr <2001:db8::a9> 0041
18:52:17.342 -> >
```
以下のような出力が見えるはずです.

コマンドの詳細はdocumentを参照してください.
将来的にはWrapperを準備するつもりです.

また, 上でslaveのアドレスを確認したら, Masterから通信することもできます.
例えば,
```
tcps 2001:db8::a9 0000
```
などとすると, ScriptにしたがってResponseがあるはずです.


