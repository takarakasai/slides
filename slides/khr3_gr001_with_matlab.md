title: khr3_gr001_with_matlab.md
date: 2016-01-13 23:30:00
tags:

# KHR3-HV & GR001 with MATLAB

<div align="center">
<img src="./slides/khr3_gr001_with_matlab/gr001_and_khr3hv.png" width="640px">
</div>

<div style="text-align: right;">
@takarakasai
</div>

---

# 1.今回の目標

Matlab(2014a)から直接ロボットを制御する.

但し(まずは）
1. Matlab本体から(Simlinkにはつながない)
1. 有線接続
1. 全関節の角度制御のみ

想定するOS
- Ubuntu Linux (14.04)
- Windows 8
- (OSX)

---

# 2.方針

以前製作したライブラリ[rsx](http://www.kohya.net/slides/index.html?rs485-with-Intel-Edison.md#5)を拡張します.

<div align="center">
<img src="./slides/khr3_gr001_with_matlab/rsx_core_structure_v0.10.png" width="320px">
</div>

---

# 3.拡張例 (C++ と Python)

<div>
<img src="./slides/khr3_gr001_with_matlab/rsx_cpp_python_structure_v0.10.png" width="320px" align="right">
<br>
既にC++とPython向けの拡張行っているので,これと同様に拡張します.
C言語で記述されたdpservoを利用する上位レイヤを用意することで実現します.
</div>

---

# 4.C言語とMatlabの連携

MatlabからC言語で実装されたライブラリを利用する手段には大きく2通り存在します.

1. 共有ライブラリをロード＆実行する汎用関数を利用する ( *1)
2. MATLABが読み込める形式の共有ライブラリ(MEXファイル)を作成する ( *2)

1.の方法は特に追加の作業なしにC言語と連携出来ますが、手順が煩雑です。
今回はより利用手順が簡単な2.の方法をとります.

*1 : http://jp.mathworks.com/help/matlab/using-c-shared-library-functions-in-matlab-.html

*2 : http://jp.mathworks.com/help/matlab/ref/mex.html

---

# 5.librsxのMEX拡張

<div>
<img src="./slides/khr3_gr001_with_matlab/rsx_mex_structure_v0.10.png" width="260px" align="right">
<br>
こんな感じになります.
<br>
<br>
MEXファイル1つに付き1関数しか実装出来ないためAPIの数だけMEXファイルを作成します.
また複数のMEXファイルが共通のハードウェアリソース（シリアルデバイス）を使用します.
このままだとAPIを実行する度にそれらの初期化等を行わなければなりません、
これを回避するためにdpsproxyという層を設けています.
</div>

---

# 6.C言語におけるMATLAB連携(1/2)

```C
/* 
 * MATLAB連携のためのヘッダファイルを読み込みます.
 */
#include "mex.h"
#include "matrix.h"

void mexFunction(int nlhs, mxArray *plhs[], int nrhs, const mxArray *prhs[]) {

    /* 
     * 引数の数と型をチェック
     * 期待値と一致しない場合にはエラーとする.
     */
    if (nrhs < 1 || !mxIsChar(prhs[0])) {
        mexErrMsgIdAndTxt(
          "dps_proxy:EINVAL",
          "dps_assign ics|rsx(string)");
        return;
    }

    /* 次ページに続く */
```

---

# 6.C言語におけるMATLAB連携(2/2)

```C
void mexFunction(int nlhs, mxArray *plhs[], int nrhs, const mxArray *prhs[]) {

    /* 前ページから */

    /*
     * 引数の解釈
     */
    int num = mxGetNumberOfElements(prhs[0]);
    char name[num + 1];
    if (mxGetString(prhs[0], name, num + 1) != 0) {
        mexErrMsgIdAndTxt("dps_proxy:ERR","");
    }

    /*
     * proxy関数の呼び出し
     */
    errno_t eno = dps_proxy_assign(name);

    /*
     * エラーチェック
     */
    if (eno != EOK) {
        mexPrintf("The input string is:  %s\n", name);
        mexErrMsgIdAndTxt("dps_proxy:ERR","");
    }

    return;
}
```

---

# 7.ビルド手順 Linux

```bash
// required package
//   -- MATLAB (MEXコンパイラのため)
> git clone https://github.com/takarakasai/rsx.git
> cd rsx
> mkdir build
> cd build
> cmake ..
> ccmake ..
 CMAKE_INSTALL_PREFIX: 任意のインストールディレクトリ
 ENABLE_MATLAB_API : ON
 HR_SERIAL_AUTO_READ_ECHO_DATA : ON (ローカルエコー有の場合)
 HR_SERIAL_AUTO_READ_ECHO_DATA : OFF (ローカルエコー無の場合)
> make
> make install
```

---

# 7.ビルド手順 Windows

```bash
// required package
//   -- MATLAB (MEXコンパイラのため)
//   -- mingw-w64    : http://mingw-w64.org/
//   -- gnumex       : http://gnumex.sourceforge.net/
//   -- git(2.7.1)   : https://git-for-windows.github.io/
//   -- cmake(3.4.1) : https://cmake.org/download/
> git clone https://github.com/takarakasai/rsx.git
> cd rsx
> . setup_mingw.sh      <<<< Windows ではこれが必要です
> mkdir build
> cd build
> cmake ..
> ccmake ..
 CMAKE_INSTALL_PREFIX: 任意のインストールディレクトリ
 ENABLE_MATLAB_API : ON
 HR_SERIAL_AUTO_READ_ECHO_DATA : ON (ローカルエコー有の場合)
 HR_SERIAL_AUTO_READ_ECHO_DATA : OFF (ローカルエコー無の場合)
> make
> make install
```

※ 必要に応じてPATHを通してください.
※ WindowsもしくはMATLABが32bitの場合にはMinGWも32bit環境を用意する必要があります
(http://www.mingw.org/).
私は試したことありませんがおそらく以下のpkgがMinGW上で必要となるはずです.
mingw-base mingw-gcc mingw32-gmp(dev) msys-ligbmp(dev)

---

# 7.起動手順 Linux/Windows

Linux の場合

```bash
> cd ${インストールディレクトリ}
> LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:../lib matlab
```

Windows の場合

```bash
MATLAB起動後にインストールディレクトリ以下のlib/mexを
MATLABのPATHに追加してください
(これであっているはず...)
```

---

# 7.1 実行手順 KHR3-HV (1関節だけ)

実行手順はポート名を除きLinux/Windowsで共通です.

```m
% サーボ通信プロトコルを指定
>> dps_assign('ics')
% サーボIDを登録
>> dps_add_servo(uint8(1))                          
id:1
% UARTポートオープン(通信速度 115200[bps])
%  Linux例   : /dev/ttyUSB0
>> dps_open('ttyUSB','0',int32(115200))             
dev:ttyUSB port:0 baudrate:115200
file open : /dev/ttyUSB0
%  Windows例 : COM0
>> dps_open('COM','0',int32(115200))             
dev:COM port:0 baudrate:115200
file open : COM0
%  Servo ON
>> dps_set_state(uint8(1), 'on')
id:01 state:on
%  30度に指定
>> dps_set_goal(uint8(1), 30.0)
```

---

# 7.2 実行手順 KHR3-HV (複数関節)

実行手順はポート名を除きLinux/Windowsで共通です.

```m
% 前ページの続き
% ID:6～10のサーボを登録
>> dps_set_servo(uint8([6, 7, 8, 9, 10]))
5 : 06 07 08 09 10
% ID:1,6～10のサーボをON
>> dps_set_states('on')
% ID:1,6～10のサーボを駆動
>> dps_set_goals([20.0, 0.0, 0.0, 0.0, 0.0, 0.0])
id:all : 20.000000 -30.000000 0.000000 0.000000 0.000000 0.000000 0.000000
```

---

# 7.3 実行手順 KHR3-HV (offset設定)

実行手順はポート名を除きLinux/Windowsで共通です.

```m
% 前ページの続き
>> dps_set_offset(uint8(1), 10.0)
id:01 oangle:10.0
>> dps_set_offsets([10.0, 0.0, 0.0, 0.0, 0.0, 0.0])
6 : 10.0 0.0 0.0 0.0 0.0 0.0
```

※ 極性の設定は今後対応予定です (mex対応するの忘れてた...).

---

# 7.1実行手順 GR001

実行手順はポート名を除きLinux/Windowsで共通です.

```m
% サーボ通信プロトコルを指定
>> dps_assign('rsx')
% 以下はKHR3-HVの場合と同じです
```

---

# 8.1 まとめ

1. MATLABでGR001とKHR3-HVを動作させる為のMEXファイルを作成しました
1. WindowsとLinuxで動作することを確認しました(OSXでも多分動く)

<div align="center">
<img src="./slides/khr3_gr001_with_matlab/rsx_structure_v0.10.png" width="380px">
</div>

---



