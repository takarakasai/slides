title: controll GR001 with Python on Edison/PC.
date: 2015-12-17 08:30:20
tags:

# Python(とEdison)でGR001を動かす

<div style="text-align: right;">
@takarakasai
</div>

---

# 今回の目標

EdisonもしくはPC上のpython環境からGR001を制御する.
これを実現するために以前C言語で作成した通信ライブラリrsxをpythonから利用できる様に拡張します.
拡張したライブラリを利用してpython環境からGR001を制御します.

※　：ライブラリの利用のみを行う場合には# 1.を飛ばしてください.

---

# 1. python用通信ライブラリ(rsxpy)のビルド/実行環境を構築

今回はEdison上でビルドします.
boost-pythonがビルドに必要で、ipythonが実行環境として便利なので
これらをインストールします.

```
> sudo apt-get install libboost-python-dev
130[MB]ほど消費されます.
> sudo apt-get install ipython
6.0[MB]ほど消費されます.
```

---

# 2. ライブラリの取得

以前作成したRS301/RS302サーボ向けの通信ライブラリを修正するため
通信ライブラリrsxを取得します.

```
> git clone git@github.com:takarakasai/rsx.git
> cd rsx
> git checkout feature/for_edison
```

---

# 3. ライブラリ内部の説明

rsxをpythonから利用するためのライブラリrsxpyに拡張します。
これにはboost_pythonを利用します.

---

## 3.1 全体構成

rsxpy向けに新たに用意したソースコードは以下のとおりです。
boost_pythonはC++のテンプレートライブラリなので、一旦C++によるラッパ(rsxpp)を作成して
boost_pythonにてpython向けのAPIを作成していきます.

```sh
./rsx/h/rsxpp/rsxpp.h                   // rsxライブラリのC++ラッパ
./rsx/python/sample/rsx_servo_state.py  // pythonによるサンプルプログラム
./rsx/python/src/rsxpy.cc               // python向けのラッパクラスの実装
./rsx/python/src/rsxpy.h                // python向けのラッパクラスのヘッダ
```

---

## 3.2 C++によるラッパクラスの作成

ここではrsxppというクラスを作成します.

```c++
-- h/rsxpp/rsxpp.h
// C言語で作成した通信ライブラリrsxを利用するためrsx.hを読み込みます.
// またシリアル通信向けのEnumを扱えるようにするためhr_serial.hも読み込みます.
extern "C" {
      #include "rsx.h"
      #include "serial/hr_serial.h"
}

// (略)
```

---

```c++
-- h/rsxpp/rsxpp.h
// (略)

// ラッパクラスを定義します.
// rsxライブラリを利用するために必要なrsx構造体を
// DEPSERVO_DECLというマクロを利用してメンバ変数として定義します.
// これをクラスのコンストラクタ内から初期化するようにします.
  class rsxpp
  {
   public:
    rsxpp(void) {
      DPSERVO_INIT(servo_inst, RSX_INIT);
      servo = (rsx*)(&servo_inst);
    }

   // (略)

   // ここにrsxライブリが用意している通信APIを一通りラップする関数を用意します.
   // 紙面の都合上全ては記載しませんが、
   // ロングパケットでデータを送信するAPIについて以下に例を示します.
   errno_t lpkt_mem_write_all
       (uint8_t id[/*num*/], uint8_t num ,
        uint8_t start_addr, uint8_t size, uint8_t data[/*size*/]) {
     ECALL(rsx_lpkt_mem_write_all(servo, id, num, start_addr, size, data));
     return EOK;
   }

   // (略)

   protected:
    DPSERVO_DECL(servo_inst, kNUM_OF_JOINTS, kMAX_PKT_SIZE, RSX_DECL);
    rsx *servo;
  }
```  

---

## 3.3 Pythonで公開するためのクラスの作成

rsxppクラスを継承したrsxpyクラスを作成します.
rsxppクラスではrsxライブラリのAPIをそのままラップしていますが、
rsxpyクラスではpythonから利用することを考えて、
pythonで扱いやすい型の変数を引数や返り値にしています.

```c++
-- python/src/rsxpy.h
// C++によるラッパクラスrsxppを利用するためのヘッダを読み込みます.
// boost_pythonを利用するためヘッダを読み込みます.
#include "rsxpp.h"
#include "boost/python.hpp"
...
class rsxpy : public dp::rsxpp<kRSXPY_NUM_OF_JOINTS, kRSXPY_MAX_PKT_SIZE> {
  ...
  // ここにrsxppライブリが用意している通信APIを一通りラップする関数を用意します.
  // 紙面の都合上全ては記載しませんが、ロングパケットでデータを送信するAPIについて以下に例を示します.
  errno_t lpkt_mem_write (bp::list &in_id, uint8_t start_addr, size_t size, bp::list &in_data) {
    GET_C_ARRAY(uint8_t, num, id/*[num]*/, in_id);
    GET_C_ARRAY_2D(uint8_t, num, size, data/*[num][size]*/, in_data);

    ECALL(base::lpkt_mem_write(id, num, start_addr, size, (uint8_t**)data));
    return EOK;
  }
  ...
}
```

---

## 3.3 boost_pythonによるAPIの公開

```c++
-- h/rsxpp/rsxpp.cc
...
// rs30xの仮想アドレスをpythonに定数として公開するため
include "mmap/rs30x.h"
...
// boost_pythonのおまじないコードです.
BOOST_PYTHON_MODULE( rsxpy ) {
  using namespace dp;

  // シリアル通信の通信レートを定数として公開します.
  boost::python::enum_<hr_baudrate>("HR_BAUDRATE")
      .value("HR_B9600"   , HR_B9600    )

  // 以下定数の公開処理が続きます.
  ...

  // ここからrsxpyクラスをpythonのrsxpyクラスとして公開します.
  boost::python::class_<rsxpy>("rsxpy")
      // rsxpyクラスの各公開関数をpythonのrsxpyクラスの関数として公開します.
      .def("open", &rsxpy::open)
      ...
      .def("lpkt_mem_write", &rsxpy::lpkt_mem_write)
      ...
      ;
}

```

---
     
# 3  . ライブラリのビルド
     
```sh
> m  kdir build
> c  make ..
> c  cmake ..
// PYTHON_API : OFF --> ON に変更します.
// gとcを押下してmakefileを生成します.
// qを押下してccmakeを抜けます.
> make
```

---

# 3. 実行

pythonのインタプリタを起動します.
ipythonだと関数や定数はタブ補完してくれます.

```sh
> cd python
> ipython
Python 2.7.6 (default, Mar 22 2014, 22:59:56) 
Type "copyright", "credits" or "license" for more information.

IPython 1.2.1 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object', use 'object??' for extra details.

In [1]: 
```

---

ipythonのインタプリタから通信APIを叩いていきます.

```python
// ライブラリを読み込んで、通信クラスをインスタンス化します.
IN[*]: import rsxpy
IN[*]: rsx = rsxpy.rsxpy()

// 通信ポートを開きます.
IN[*]: rsx.open("ttyMFD", "1", rsxpy.HR_BAUDRATE.HR_B460800, rsxpy.HR_PARITY.HR_PAR_NONE)
// PCから直接GR001を制御する場合には下記の様に第1、2引数を読み替えてください.
IN[*]: rsx.open("ttyUSB", "0", rsxpy.HR_BAUDRATE.HR_B460800, rsxpy.HR_PARITY.HR_PAR_NONE)

IN[*]: ids = range(1,21)

// サーボをONにします.
IN[*]: rsx.lpkt_mem_write_all(ids, rsxpy.RSX_RS30X_MEM_ADDR.RSX_RAM_TRQ_ENABLE, [1])

// 各関節の角度を取得します.
IN[*]: ang = list()
IN[*]: for id in ids :
IN[*]:   ang.append(rsx.lpkt_mem_read_int16(id, rsxpy.RSX_RS30X_MEM_ADDR.RSX_RAM_PRESENT_POS_L, 1)[0])
IN[*]:

// 1番と4番関節の角度を30度変化させます.
IN[*]: ang[1] += 30 * 10;
IN[*]: ang[4] += 30 * 10;
IN[*]: rsx.lpkt_mem_write_int16(ids, rsxpy.RSX_RS30X_MEM_ADDR.RSX_RAM_GOAL_POS_L, 1, ang)

// サーボをOFFします.
IN[*]: rsx.lpkt_mem_write_all(ids, rsxpy.RSX_RS30X_MEM_ADDR.RSX_RAM_TRQ_ENABLE, [0])

```

---

