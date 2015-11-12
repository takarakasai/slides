title: xenomai-3.0 on ubilinux with Intel Edison
date: 2015-11-01 05:43:20
tags:

# Intel EdisonでGR001とRS485で通信

<div align="center">
<img src="./slides/rs485-with-Intel-Edison/gr001_rear.jpg" width="640px">
</div>

<div style="text-align: right;">
@takarakasai
</div>

---

# 1.今回の目標

- Intel EdisonでGR001とRS485で通信する.

- 複数の構成が考えられるので代表的なものを試す.

- できれば通信時間等の比較をしてみる.

---

# 2.実施する構成

1. USB-RS232C変換ケーブル + RS232C-RS485変換コネクタを使用する

2. USB-RS485変換コネクタを使用する

3. RS232C-RS485変換ICを使用する.

---

# 3.想定する環境

## break out board

- henry board

## OS

- ubilinux + xenomai3.0.0

## library

- cmake
- cmake-ncurses-gui

---

# 4.コードのビルドと実行

全身のサーボの目標角度の更新と現在角度の取得を繰り返し行い。
その間に消費される時間を測定します.

```bash
> mkdir work
> cd work
> git clone git@github.com:takarakasai/rsx.git
> mkdir -p rsx/build
> cd rsx/build
> camke ..
> ccmake ..
   ここで4.に示すコンフィグを行います.
> make
> ./sample/rsx_latency_test tty*** *
   ここで4.に示すttyUSB/ttyMFD 0/1の値を指定します.
```

---

# 4.実施形態

---

## 4.1 実施形態１ (USB-RS232C + RS232C-RS485)

```ascii
  +--------+  USB  +------------+ RS232C +------------+ RS485 +-------+
  | Edison |=======| USB-RS232C |========|RS232C-RS485|=======| GR001 |
  +--------+       +------------+        +------------+       +-------+
```

<div align="center">
<img src="./slides/rs485-with-Intel-Edison/ttyUSB2rs232c2rs485.jpg" width="320px">
</div>

---

## 4.1.1 サーボON中の現在角度取得 (Echo:On)

- 1軸あたり約3.0[msec]

```bash
file open : /dev/ttyUSB0
 auto echo read latency : 000000000001996.318[usec]
           read latency : 000000000000899.985[usec]
Model Number L:10 H:30
Firmware Version:05
 auto echo read latency : 000000000001928.862[usec]
           read latency : 000000000000941.456[usec]
 auto echo read latency : 000000000001975.050[usec]
           read latency : 000000000000959.415[usec]
 auto echo read latency : 000000000001979.356[usec]
           read latency : 000000000000939.320[usec]
 auto echo read latency : 000000000002017.731[usec]
           read latency : 000000000000928.883[usec]
 auto echo read latency : 000000000001944.446[usec]
           read latency : 000000000001006.780[usec]
 auto echo read latency : 000000000001927.463[usec]
           read latency : 000000000000954.515[usec]
 auto echo read latency : 000000000002015.670[usec]
           read latency : 000000000000936.443[usec]
 auto echo read latency : 000000000001963.297[usec]
```

---

## 4.1.1 サーボON中の現在角度取得 (Echo:Off)

- 1軸あたり約3.0[msec], Echoなしとあまり変わらない。

```bash
file open : /dev/ttyUSB0
           read latency : 000000000002938.702[usec]
Model Number L:10 H:30
Firmware Version:05
           read latency : 000000000002889.709[usec]
           read latency : 000000000002944.792[usec]
           read latency : 000000000002939.965[usec]
           read latency : 000000000002937.811[usec]
           read latency : 000000000002943.545[usec]
           read latency : 000000000002945.761[usec]
           read latency : 000000000002950.890[usec]
           read latency : 000000000002932.800[usec]
           read latency : 000000000002945.314[usec]
           read latency : 000000000002970.654[usec]
           read latency : 000000000002937.247[usec]
           read latency : 000000000002954.433[usec]
           read latency : 000000000002910.236[usec]
           read latency : 000000000002951.315[usec]
           read latency : 000000000002956.522[usec]
           read latency : 000000000002945.080[usec]
```

---

## 4.1.2 目標角度更新+現在角度取得 (Echo:On)

- 1回あたり約2200[msec],遅い...

```bash
root@ubilinux:/home/kasai/rsx/build# ./sample/rsx_latency_test ttyUSB 0
file open : /dev/ttyUSB0
Model Number L:10 H:30
Firmware Version:05
     write read latency : 000000002209018.433[usec]
     write read latency : 000000002209550.048[usec]
     write read latency : 000000002215498.369[usec]
     write read latency : 000000002210337.490[usec]
     write read latency : 000000002214434.746[usec]
     write read latency : 000000002210338.889[usec]
     write read latency : 000000002208474.912[usec]
     write read latency : 000000002213613.604[usec]
     write read latency : 000000002207402.759[usec]
     write read latency : 000000002223359.310[usec]
     write read latency : 000000002214373.243[usec]
     write read latency : 000000002209397.254[usec]
     write read latency : 000000002215453.894[usec]
     write read latency : 000000002221349.023[usec]
     write read latency : 000000002213387.381[usec]
     write read latency : 000000002226612.283[usec]
     write read latency : 000000002229343.842[usec]
     write read latency : 000000002207545.825[usec]
     write read latency : 000000002218341.943[usec]
     write read latency : 000000002221342.090[usec]
```

---

## 4.1.2 目標角度更新+現在角度取得 (Echo:Off)

- 1回あたり約66[msec], Echoなしとだいぶ変わる.

```bash
root@ubilinux:/home/kasai/rsx/build# ./sample/rsx_latency_test ttyUSB 0 
file open : /dev/ttyUSB0
Model Number L:10 H:30
Firmware Version:05
     write read latency : 000000000066007.243[usec]
     write read latency : 000000000065416.549[usec]
     write read latency : 000000000065517.149[usec]
     write read latency : 000000000065456.895[usec]
     write read latency : 000000000065402.658[usec]
     write read latency : 000000000065359.403[usec]
     write read latency : 000000000065491.657[usec]
     write read latency : 000000000065477.599[usec]
     write read latency : 000000000065335.568[usec]
     write read latency : 000000000065364.201[usec]
     write read latency : 000000000065544.980[usec]
     write read latency : 000000000065445.083[usec]
     write read latency : 000000000065318.323[usec]
     write read latency : 000000000065438.933[usec]
     write read latency : 000000000065735.271[usec]
     write read latency : 000000000065654.119[usec]
     write read latency : 000000000065327.452[usec]
     write read latency : 000000000065310.521[usec]
     write read latency : 000000000065388.858[usec]
     write read latency : 000000000065337.072[usec]
----- end -----
```

---

## 4.2 実施形態2 (USB-RS485)

```ascii
  +--------+  USB  +----------+ RS485 +-------+
  | Edison |=======| USB-RS485|=======| GR001 |
  +--------+       +----------+       +-------+
```

<div align="center">
<img src="./slides/rs485-with-Intel-Edison/ttyUSB2rs485.jpg" width="320px">
</div>

---

## 4.2.0 実施形態2 (USB-RS485) 準備

- 今回使用する変換コネクタはCH340+MAX485の構成なので,ch341なドライバが必要.
- Edison標準のLinuxイメージにはこれがないのでドライバの導入が必要.
- 標準でサポートしているのでカーネルコンフィグをいじってドライバをビルドする.[参考](file:///home/kasai/study/slide/slides/sample.html?xenomai-3-0-on-ubilinux-with-Intel-Edison.md#11)


1. インストール場所はこんな感じ

```bash
hroot@ubilinux:home/kasai/rsx/build# find /lib/modules/3.10.17-yocto-standard | grep ch341.ko$
/lib/modules/3.10.17-yocto-standard/kernel/drivers/usb/serial/ch341.ko
hroot@ubilinux:home/kasai/rsx/build# cat /lib/modules/3.10.17-yocto-standard/modules.dep | grep ch341
kernel/drivers/usb/serial/ch341.ko:
```

2. ドライバをロードすると/dev/ttyUSB*として認識する

```bash
root@ubilinux:/home/kasai/rsx/build# dmesg | grep ch341
//ロードしていないので何も表示されない
root@ubilinux:/home/kasai/rsx/build# modprobe ch341
root@ubilinux:/home/kasai/rsx/build# dmesg | grep ch341
[ 4594.146533] usbcore: registered new interface driver ch341
[ 4594.146684] usbserial: USB Serial support registered for ch341-uart
[ 4594.146773] ch341 1-1:1.0: ch341-uart converter detected
[ 4594.149669] usb 1-1: ch341-uart converter now attached to ttyUSB0
```

---

## 4.2.1 サーボON中の現在角度取得 (Echo:Off)

- 1軸あたり40.0-50.0[msec]程度の時間が消費されるようです. 
- ばらつきがあります.

```bash
file open : /dev/ttyUSB0
           read latency : 000000000004025.258[usec]
Model Number L:10 H:30
Firmware Version:05
           read latency : 000000000004837.597[usec]
           read latency : 000000000004894.456[usec]
           read latency : 000000000004113.485[usec]
           read latency : 000000000004768.715[usec]
           read latency : 000000000004956.296[usec]
           read latency : 000000000004102.266[usec]
           read latency : 000000000004829.400[usec]
           read latency : 000000000004925.636[usec]
           read latency : 000000000004913.047[usec]
           read latency : 000000000004939.229[usec]
           read latency : 000000000004973.907[usec]
           read latency : 000000000004061.789[usec]
           read latency : 000000000004838.127[usec]
           read latency : 000000000004946.335[usec]
           read latency : 000000000004037.351[usec]
           read latency : 000000000004833.191[usec]
```

---

## 4.2.2 目標角度更新+現在角度取得 (Echo:Off)

- 1軸あたり100[msec]程度の時間が消費されるようです.

```bash
root@ubilinux:/home/kasai/rsx/build# ./sample/rsx_latency_test ttyUSB 0
file open : /dev/ttyUSB0
Model Number L:10 H:30
Firmware Version:05
     write read latency : 000000000099950.036[usec]
     write read latency : 000000000098789.981[usec]
     write read latency : 000000000099444.884[usec]
     write read latency : 000000000098239.896[usec]
     write read latency : 000000000099361.530[usec]
     write read latency : 000000000100248.693[usec]
     write read latency : 000000000100436.427[usec]
     write read latency : 000000000101202.667[usec]
     write read latency : 000000000100670.694[usec]
     write read latency : 000000000100652.687[usec]
     write read latency : 000000000101576.298[usec]
     write read latency : 000000000100529.411[usec]
     write read latency : 000000000101556.370[usec]
     write read latency : 000000000101491.546[usec]
     write read latency : 000000000100395.651[usec]
     write read latency : 000000000099471.337[usec]
     write read latency : 000000000095471.131[usec]
     write read latency : 000000000099132.585[usec]
     write read latency : 000000000101567.170[usec]
     write read latency : 000000000101641.469[usec]
----- end ----- 
```

---

## 4.3 実施形態3 (RS232C-RS485)

```ascii
  +--------+ RS232C +------------+ RS485 +-------+
  | Edison |========|RS232C-RS485|=======| GR001 |
  +--------+        +------------+       +-------+
```

<div align="center">
<img src="./slides/rs485-with-Intel-Edison/ttyMFD2rs485_frisk.jpg" width="320px">
<img src="./slides/rs485-with-Intel-Edison/ttyMFD2rs485.jpg" width="320px">
<br>
フリスクケースあり (左)
フリスクケースなし (右)
</div>

---

## 4.3 実施形態2´ (RS232C-RS485)

```ascii
  +--------+
  | Edison |
  +--------+
      || uart 
  +--------+ RS232C +------------+ RS485 +-------+
  |自作基板|========|RS232C-RS485|=======| GR001 |
  +--------+        +------------+       +-------+
```

<div align="center">
<img src="./slides/rs485-with-Intel-Edison/ttyMFD2rs485_1board_1.jpg" width="320px">
=>
<img src="./slides/rs485-with-Intel-Edison/ttyMFD2rs485_1board_2.jpg" width="320px">
</div>

---

## 4.3.1 サーボON中の現在角度取得

- 1軸あたり2.5[msec]程度の時間が消費されるようです.

```bash
file open : /dev/ttyMFD1
 auto echo read latency : 000000000002568.977[usec]
           read latency : 000000000000154.489[usec]
Model Number L:10 H:30
Firmware Version:05
 auto echo read latency : 000000000002389.956[usec]
           read latency : 000000000000153.327[usec]
 auto echo read latency : 000000000002391.869[usec]
           read latency : 000000000000150.943[usec]
 auto echo read latency : 000000000002416.129[usec]
           read latency : 000000000000161.699[usec]
 auto echo read latency : 000000000002291.760[usec]
           read latency : 000000000000161.488[usec]
 auto echo read latency : 000000000002269.892[usec]
           read latency : 000000000000165.285[usec]
 auto echo read latency : 000000000002278.397[usec]
           read latency : 000000000000163.622[usec]
 auto echo read latency : 000000000002203.704[usec]
           read latency : 000000000000164.331[usec]
 auto echo read latency : 000000000002218.209[usec]
```

---

## 4.3.2 目標角度更新＋現在角度取得

- 1回あたり57[msec]程度の時間が消費されるようです.

```bash
root@ubilinux:/home/kasai/rsx/build# ./sample/rsx_latency_test ttyMFD 1
file open : /dev/ttyMFD1
Model Number L:10 H:30
Firmware Version:05
     write read latency : 000000000056354.259[usec]
     write read latency : 000000000056687.984[usec]
     write read latency : 000000000056654.634[usec]
     write read latency : 000000000056597.344[usec]
     write read latency : 000000000056777.629[usec]
     write read latency : 000000000056968.642[usec]
     write read latency : 000000000056450.153[usec]
     write read latency : 000000000056666.476[usec]
     write read latency : 000000000056631.691[usec]
     write read latency : 000000000056884.130[usec]
     write read latency : 000000000056841.407[usec]
     write read latency : 000000000056458.870[usec]
     write read latency : 000000000056554.745[usec]
     write read latency : 000000000056651.169[usec]
     write read latency : 000000000056691.800[usec]
     write read latency : 000000000056875.255[usec]
     write read latency : 000000000056446.435[usec]
     write read latency : 000000000056648.150[usec]
     write read latency : 000000000056814.218[usec]
     write read latency : 000000000057273.610[usec]
```

---

## 5 結果まとめ

<table border=1 align=center>
 <tr><th></th><th>USB-RS232-RS485</th><th>USB-RS485</th><th>uart-RS232-RS485</th></tr>
 <tr><td>              現在角度取得</td><td> 3.0[msec]</td><td>40-50.0[msec]</td><td> 2.5[msec]</td></tr>
 <tr><td>目標角度更新と現在角度取得</td><td>67.0[msec]</td><td>  100.0[msec]</td><td>57.0[msec]</td></tr>
</table>

---

## おわり

<div align="center">
<img src="./slides/rs485-with-Intel-Edison/makerbot_replicator2x.jpg" width="320px">
<img src="./slides/rs485-with-Intel-Edison/gr001_front.jpg" width="320px">
</div>


