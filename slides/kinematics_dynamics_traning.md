class: center, middle

title: controll GR001 with Python on Edison/PC.
date: 2015-12-17 08:30:20
tags:

# 順運動学と順動力学を実践する

<div style="text-align: right;">
@takarakasai
</div>

---

# 今回の目標

ヒューマノイドロボットでは第2章、第3章においてロボット（剛体他関節体）の
運動学と動力学について触れている。
今回は例として6自由度のシリアルリンクアームに関して
運動学と動力学を活用した動力学シミュレータ(仮)を作成してみる。

---

# 1. 実行環境

ロボット実機の制御に生かすことを考慮して,使用言語はpythonとする.

## 運動学と動力学演算に必要なもの

- numpy

## シミュレータ環境の特に描画に必要なもの

- PyOpenGL
- PyFTGL

---

# 2.順動力学

## 順運動学に関する方程式

漸化式(1)を計算してリンク先端の位置を求めます.
回転行列 R を算出するには関節の角度と回転方向を考慮します.

`\(
\begin{eqnarray}
P_i &=& R_i \cdot p_i + P_{i-1}  \\
&& p_i : リンク座標系におけるリンクi先端の位置 \nonumber \\
&& P_i : ワールド座標系におけるリンクi先端の位置 \nonumber \\
&& R : リンク座標系を親リンク座標系に変換する回転行列 \nonumber \\
\end{eqnarray}
\)`

---

# 3.順動力学

## 順動力学に関する方程式

関節空間における運動方程式(2)か関節の角加速度を求めます.
慣性行列はより先端のリンク群の慣性行列と姿勢を考慮します.

$$
\begin{eqnarray}
\tau &=& H \ddot q + C - J^{T}f \\\
&& \tau : 関節内力 \nonumber \\\
&& H : 関節空間における慣性行列 \nonumber \\\
&& \ddot q : 関節の角加速度 \nonumber \\\
&& C : 重力，コリオリ力,遠心力項 \nonumber \\\
&& J : 関節空間と操作空間とを関連付けるヤコビ行列 \nonumber \\\
&& f : 操作空間においてリンクに作用する力 \nonumber \\\
\end{eqnarray}
$$

---

# 4. 動作確認

すべての演算をpython(numpy)を利用して行っているため動作はあまり高速ではありません.
1周期あたり4.0[msec]程度の時間がかかっています.

<table align=center><tr><td>
<video controls align=center>
 <source src="slides/kinematics_dynamics_traning/rkd_tl_test_20160310.mp4" type="video/mp4">
 <source src="slides/kinematics_dynamics_traning/rkd_tl_test_20160310.flv" type="video/flv">
 <p>動画を再生するにはvideoタグをサポートしたブラウザが必要です。</p>
</video>
</td></tr></table>

