---
title: PIDコントローラによるモータの速度制御の理論的解析
tags:
  - モーター制御
  - PID制御
private: false
updated_at: '2024-07-08T10:28:58+09:00'
id: 1b758a49e24640678f6d
organization_url_name: null
slide: false
ignorePublish: false
---
# 0. はじめに
~~この記事は未完成ですが、あと少しなので、ひとまず公開して意見が欲しいから公開しました。~~
更新の予定は全くないです。すみません。

# 1. Motorの基礎計算
## 1.1 伝達関数の導出
モータの機械・電気的特性は以下の二つの支配方程式によって表現される。

```math
\left\{\begin{align*}
e &= Ri+L\frac{di}{dt}+k_e\omega\\
k_Ti &=T_L+B\omega+J\frac{d\omega}{dt}
\end{align*}\right. \tag{1-1}
```

この二つの支配方程式をラプラス変換すると下式となる。

```math
\left\{\begin{align*}
E&=RI+sLI+k_e\Omega\\
k_TI&=\frac{T_L}{s}+B\Omega+sJ\Omega
\end{align*}\right. \tag{1-2}
```

ここで、各機構は以下の表の通りである。

|記号||
|---|---|
|$R$|電機子抵抗|
|$L$|電機子インダクタンス|
|$T_L$|負荷トルク（外乱）|
|$B$|粘性係数|
|$J$|慣性モーメント|
|$e=\mathcal{L}^{-1}[E]$|印加電圧|
|$i=\mathcal{L}^{-1}[I]$|電流|
|$\omega=\mathcal{L}^{-1}[\Omega]$|角速度|

(1-2)式から速度への伝達関数を計算する。

```math
\begin{align*}
\Omega(s) &= \frac{1}{k_e}E -\frac{R+sL}{k_e}I \\
&= \frac{k_T}{B+sJ}I-\frac{1}{s(B+sJ)}T_L \\
(\frac{k_T}{B+sJ}+\frac{R+sL}{k_e})\Omega(s)&= \frac{k_T}{B+sJ}\frac{1}{k_e}E-\frac{R+sL}{k_e}\frac{1}{s(B+sJ)}T_L\\
(k_Tk_e+(R+sL)(B+sJ))\Omega(s)&= k_TE - (R+sL)T_L \\
\Omega(s)&=\frac{k_T}{k_Tk_e+(R+sL)(B+sJ)}E-\frac{R+sL}{k_Tk_e+(R+sL)(B+sJ)}T_L \tag{1-3}
\end{align*}
```

(1-3)式は電圧と負荷トルクからの二つの伝達関数を示している。

```math
\Omega(s)=G_{E}(s)E(s)+G_{T_L}(s)T_L(s) \tag{1-4}
```

モータから速度のみを取得し速度制御を行う場合、モータの入力は電圧から速度への電圧関数を調べればよい。計算の都合、負荷トルク$T_L$は外乱として取り扱うこととする。よって、モータの伝達関数$G_M(s)$は

```math
\begin{align*}
G_M(s)=\frac{\Omega}{E}&=\frac{k_T}{(R+sL)(B+sJ)+k_ek_T} \\
&=\frac{k_T}{LJs^2+(BL+JR)s+RB+k_ek_T} \tag{1-5}
\end{align*}
```

この伝達関数$G_M(s)$はひとまず二次遅れ系であることがわかる。

## 1.2 一般的なモータの理解と基礎式
前章でモータの特性は二次遅れ系であることを示した((1-5)式)。一方で簡易的なモータ制御では、モータは一次遅れ系で扱われることがほとんどである。実際モータのステップ応答を確認すると、一次遅れ系のような応答を示す。この要因について解説する。
モータのパラメータについてまとめる。

|パラメータ||オーダー|
|---|---|---|
|R|電機子抵抗|数百$m\Omega$から数$\Omega$|
|L|電機子インダクタンス|数$\mu H$から数$mH$|
|B|粘性係数|ほとんど計測できない|
|J|慣性モーメント||

これらのパラメータのうち、ほとんどのモータにおいて$L$と$B$は非常に小さいことが知られている（当然、大きいものもある。Bはモータ単体では小さいが、アプリケーションによって変わる）。これらを0と置く（$L\rightarrow0$,$B\rightarrow0$）とモータの伝達関数((1-5)式)は

```math
G'_M(s)=\frac{\frac{1}{k_e}}{\frac{JR}{k_Tk_e}s+1}\tag{1-6}
```

と書き換わる。この式は前の説明通り、時定数$\frac{JR}{k_Tk_e}$、飽和速度$\frac{1}{k_e}E$の一時遅れ系であることが示された。この時定数$\tau=\frac{JR}{k_Tk_e}$がモータによる機構設計でよく用いられる時定数である。移行のモータの伝達関数は(1-6)式を用いる。

```math
G_M(s)=\frac{A}{\tau s+1},\;\left(A=\frac{1}{k_e},\tau=\frac{JR}{k_Tk_e}\right) \tag{1-7}
```


# 2. PIDの伝達関数
PID制御はいくつかのバリエーションがあるが、ここでは最も一般出来であると考える

```math
y=K_pe+K_i\int{edt}+K_d\frac{de}{dt} \tag{2-1}
```

ここで、$e$は目標値と現在値の偏差であり、$K_p,K_i,K_d$はそれぞれ比例・積分・微分制御のゲインである。
PID((2-1)式)の伝達関数$G_{c}(s)$は下式によって求められる。

```math
G_{c}(s)=\frac{K_ps+K_i+K_ds^2}{s} \tag{2-2}
```

また、もう一つ次のPID制御式を導入する

```math
y=K\left(e+\frac{1}{T_i}\int{edt}+T_d\frac{de}{dt}\right) \tag{2-3}
```

(2-3)式は微積項の単位が合っていないため、各ゲインを時間の単位を持つパラメータに変更してあらわしたものである。この式は初学者向けの資料ではあまり見ないが、後の解析において非常に使いやすいのでここで導入する。各パラメータの換算式は下記のとおりである。

```math
\begin{align*}
\left\{
\matrix{Kp&=K\\K_i&=\frac{K}{T_i}\\K_d&=KT_d}
\right.\tag{2-4}
\end{align*}
```

# 3. 単一速度制御の伝達関数
電流制御を行わない単純な速度制御を考える。速度指令値を入力しPIDコントローラが電圧(duty)を決定し、モータに入力してモータが回転し、速度をフィードバックしてコントローラに戻るクローズドループ制御を行うモノとする。
計算の都合、フィードバックの伝達関数$H(s)=1$とする。これらより、閉ループ伝達関数$G(s)$は以下のように求められる。

```math
\begin{align*}
    G(s)&=\frac{G_M(s)G_c(s)}{1+G_M(s)G_c(s)} \\
    &=\frac{A(K_ps+K_i+K_ds^2)}{s(\tau s+1)+A(K_ps+K_i+K_ds^2)} \\
    &=\frac{A(K_ds^2+K_ps+K_i)}{(AK_d+\tau)s^2+(AK_p+1)s+AK_i} \tag{3-1}
\end{align*}
```

フルビッツの安定判別法を用いてこの伝達関数$G(s)$を解析すると、すくなくともすべてのゲインが0以上であれば安定することが示される。


# 4. PID制御されたモータのステップレスポンス
速度指示値にステップ

```math
\begin{align*}
\omega_{ref}=\left\{\matrix{0,(t<0)\\1,(t>=0)}\right.\\
\Omega_{ref}(s)=\frac{1}{s}
\end{align*}\tag{4-1}
```

を入力したときのモータ((3-1)式)の応答（ステップレスポンス）について考える。

```math
\begin{align*}
\Omega &= \frac{A(K_ds^2 + K_ps+K_i)}{(AK_d +\tau) s^2+(AK_p+1)s+AK_i}\frac{1}{s} \\
&=\frac{1}{s}-\frac{1}{\tau_d}\frac{\tau s+1}{s^2+\frac{AK_p+1}{\tau_d}s+\frac{AK_i}{\tau_d}} &\left(\tau_d=AK_d+\tau\right)\\
&=\frac{1}{s}-\frac{1}{\tau_d}\left(\frac{c_1}{s-\lambda_1}+\frac{c_2}{s-\lambda_2}\right)\\
\end{align*}\tag{4-2}
```

(4-2)式のパラメータ$\lambda_1\lambda_2,c_1,c_2$について決定する。

```math
\begin{align*}
\lambda_1+\lambda_2&=-\frac{AK_p+1}{\tau_d}=-\tau_p &,\; \lambda_1\lambda_2&=\frac{AK_i}{\tau_d}=\tau_i \\
c_1+c_2&=\tau &,\;c1\lambda_2+c_2\lambda_1&=-1\\
\end{align*}
```

```math
\begin{align*}
\lambda_1 &= \frac{-\tau_p+\sqrt{\tau_p^2-4\tau_i}}{2}&,\;\lambda_2&=\frac{-\tau_p-\sqrt{\tau_p^2-4\tau_i}}{2}
\end{align*}
```

```math
\begin{align*}
c_1&=\frac{1+\lambda_1\tau}{\sqrt{\tau_p^2-4\tau_i}}=\frac{(1-\frac{1}{2}\tau\tau_p)+\frac{1}{2}\tau\sqrt{\tau_p^2-4\tau_i}}{\sqrt{\tau_p^2-4\tau_i}}=\frac{1-\frac{1}{2}\tau\tau_p}{\sqrt{\tau_p^2-4\tau_i}}+\frac{\tau}{2}\\
c_2&=-\frac{1+\lambda_2\tau}{\sqrt{\tau_p^2-4\tau_i}}=-\frac{(1-\frac{1}{2}\tau\tau_p)-\frac{1}{2}\tau\sqrt{\tau_p^2-4\tau_i}}{\sqrt{\tau_p^2-4\tau_i}}=-\frac{1-\frac{1}{2}\tau\tau_p}{\sqrt{\tau_p^2-4\tau_i}}+\frac{\tau}{2}\\
\end{align*}\tag{4-3}
```

(4-2)式を逆ラプラス変換を行うと

```math
\begin{align*}
\omega&=1-\frac{1}{\tau_d}\left(c_1e^{\lambda_1t}+c_2e^{\lambda_2t}\right)\\
&=1-\frac{1}{\tau_d}\exp{\left(-\frac{\tau_pt}{2}\right)}\left(c_1\exp{\frac{\sqrt{\tau_p^2-4\tau_i}}{2}t}+c_2\exp{-\frac{\sqrt{\tau_p^2-4\tau_i}}{2}t}\right)\\
&=1-\frac{1}{\tau_d}\exp{\left(-\frac{\tau_pt}{2}\right)}\left(\frac{\tau}{2}\left(\exp{\frac{\sqrt{\tau_p^2-4\tau_i}}{2}t}+\exp{-\frac{\sqrt{\tau_p^2-4\tau_i}}{2}t}\right)+\\ \frac{1-\frac{1}{2}\tau\tau_p}{\sqrt{\tau_p^2-4\tau_i}}\left(\exp{\frac{\sqrt{\tau_p^2-4\tau_i}}{2}t}-\exp{-\frac{\sqrt{\tau_p^2-4\tau_i}}{2}t}\right)\right)\\
&=1-\frac{1}{\tau_d}\exp{\left(-\frac{\tau_pt}{2}\right)}\left(\tau\cosh{\frac{\sqrt{\tau_p^2-4\tau_i}}{2}t}+\frac{2-\tau\tau_p}{\sqrt{\tau_p^2-4\tau_i}}\sinh{\frac{\sqrt{\tau_p^2-4\tau_i}}{2}t}\right)\tag{4-4}\\
\end{align*}
```

(4-4)式に(4-3)式で省略した変数を戻す

```math
\begin{align*}
\omega=1-\frac{1}{AK_d+\tau}\exp{\left(-\frac{AK_p+1}{2\left(AK_d+\tau\right)}t\right)}\left( \tau\cosh{\left(\frac{\sqrt{\left(\frac{AK_p+1}{AK_d+\tau}\right)^2-4\left(\frac{AK_i}{AK_d+\tau}\right)}}{2}t\right)}+\\
\frac{2-\tau\frac{AK_p+1}{AK_d+\tau}}{\sqrt{\left(\frac{AK_p+1}{AK_d+\tau}\right)^2-4\left(\frac{AK_i}{AK_d+\tau}\right)}}\sinh{\left(\frac{\sqrt{\left(\frac{AK_p+1}{AK_d+\tau}\right)^2-4\left(\frac{AK_i}{AK_d+\tau}\right)}}{2}t\right)}\right)
\end{align*}\tag{4-5}
```

以上(4-5)式より、PID制御されたモータのステップレスポンスの時間関数が得られた。次の項からはこのステップレスポンスを解析し、各ゲインが制御にどのような影響を与えているのかを調べる。

## 4.1 比例制御のステップレスポンスの解析
(4-5)式は複雑なので、前処理として微分を無効にする。

```math
\begin{align*}
\omega=1-\exp{\left(-\frac{AK_p+1}{2\tau}t\right)}\left( \cosh{\left(\frac{\sqrt{\left(\frac{AK_p+1}{\tau}\right)^2-4\frac{AK_i}{\tau}}}{2}t\right)}+\\
\frac{1-AK_p}{\tau\sqrt{\left(\frac{AK_p+1}{\tau}\right)^2-4\frac{AK_i}{\tau}}}\sinh{\left(\frac{\sqrt{\left(\frac{AK_p+1}{\tau}\right)^2-4\frac{AK_i}{\tau}}}{2}t\right)}\right)
\end{align*}\tag{4-6}
```

続いて積分を無効にして比例制御によるステップレスポンスの式を得る

```math
\begin{align*}
\omega = 1-\exp{\left(-\frac{AK_p+1}{2\tau}t\right)}\left(\cosh{\left(\frac{AK_p+1}{2\tau}t\right)}+\frac{(1-AK_p)}{AK_p+1}\sinh{\left(\frac{AK_p+1}{2\tau}t\right)}\right)
\end{align*}\tag{4-7}
```

ここで

```math
\begin{align*}
\exp{(-x)}\cosh{(x)}&=\exp{(-x)}\frac{\exp{(x)}+\exp{(-x)}}{2}=\frac{1+\exp{(-2x)}}{2}\\
\exp{(-x)}\sinh{(x)}&=\exp{(-x)}\frac{\exp{(x)}-\exp{(-x)}}{2}=\frac{1-\exp{(-2x)}}{2}
\end{align*}\tag{4-8}
```

であるから

```math
\begin{align*}
\omega &= 1-\frac{1}{2}\left(1+\exp{\left(-\frac{AK_p+1}{\tau}t\right)}+\frac{1-AK_p}{AK_p+1}\left(1-\exp{\left(-\frac{AK_p+1}{\tau}t\right)}\right)\right)\\
&=\frac{AK_p}{AK_p+1}\left(1-\exp{\left(-\frac{AK_p+1}{\tau}t\right)}\right)\tag{4-9}
\end{align*}
```

と求まる。これは、一時遅れ系の比例制御の式と完全に同一である。比例制御での時定数は

```math
\begin{align*}
\tau_c=\frac{\tau}{AK_p+1} \tag{4-10}
\end{align*}
```

また、平衡時の定常偏差($t\to\infty$)は

```math
\begin{align*}
\mathrm{error} = 1 - \omega(\infty)=1-\frac{AK_p}{AK_p+1}=\frac{1}{K_p/k_e+1} \tag{4-11}
\end{align*}
```

である。
制御後の時定数は元の機構の時定数より$\frac{1}{AK_p+1}$倍されていることがわかる。比例制御によって、オープンループ制御より高速に動作させることが可能であることを示しているが、当然、ドライバに入力した電圧によって、制限されることには注意されたい。最終的には、ドライバに入力した電圧をそのままモータにかけた加速度より早く応答することは不可能である。<b>あくまで、目的の速度に平衡する電圧をステップで掛けた場合と、その目標速度を入力した速度制御の時定数の比である</b>。

また、$A$とは(1-7)式で省略された$A=1/k_e$であることを思い出す。$K_p$が有限の値である限り定常偏差が現れKpが小さいほど定常偏差が大きくなってしまうことがわかる。(4-11)式より、$K_p$が$k_e$より十分に大きいときは定常偏差が十分に小さいことが見込める。

## 4.2積分制御のステップレスポンスの解析

積分制御の場合について考える。PI制御式((4-6)式）を見ると、平方根があり、この中の値$D$が正か負かに応じて変化する。

```math
\begin{align*}
D&=\left(\frac{AK_p+1}{\tau}\right)^2-4\frac{AK_i}{\tau} \tag{4-12}
\end{align*}
```

この三つの区間に分かれる。

```math
\begin{align*}
\omega&=\left\{
    \matrix{
       1-\exp{\left(-\frac{AK_p+1}{2\tau}t\right)}\left( \cosh{\left(\frac{\sqrt{D}}{2}t\right)}+\frac{1-AK_p}{\tau\sqrt{D}}\sinh{\left(\frac{\sqrt{D}}{2}t\right)}\right) &,(D>0)\\
        1-\exp{\left(-\frac{AK_p+1}{2\tau}t\right)}\left(1+\frac{1}{2}(1-AK_p)t\right) &,(D=0)\\
        1-\exp{\left(-\frac{AK_p+1}{2\tau}t\right)}\left( \cos{\left(\frac{\sqrt{-D}}{2}t\right)}+\frac{1-AK_p}{\tau\sqrt{-D}}\sin{\left(\frac{\sqrt{-D}}{2}t\right)}\right) &,(D<0)\\
    }
\right.
\end{align*}\tag{4-13}
```

### $D$について
$D$が0より小さくなる条件について考える。

```math
\begin{align*}
D=\left(\frac{AK_p+1}{\tau}\right)^2-4\frac{AK_i}{\tau}&<0 \\
K_i&>\frac{\tau}{4A}\left(\frac{AK_p+1}{\tau}\right)^2 \\
&> \frac{\tau}{4A\tau_c^2}\\
\end{align*}
```

よって、積分ゲインは、制御対象単体の時定数$\tau$と比例制御での時定数$\tau_c$の二乗の比に比例していることがわかる。しかし、これでもわかりずらい。ここで、PID制御式を(2-1)式から(2-3)式に変更すると

```math
\begin{align*}
K_i = \frac{K}{T_i} &> \frac{\tau}{4A}\left(\frac{AK_p+1}{\tau}\right)^2 \\
T_i&<\frac{4\tau}{A}\left(\frac{AK}{AK+1}\right)^2
\end{align*}
```

ここで、ゲイン$AK$が1より十分に大きい場合、$AK/(AK+1)\to1$であるから

```math
T_i<\frac{4\tau}{AK}\fallingdotseq4\tau_c
```

であるとき、$D<0$であることがわかる。$AK\gg1$という制約が成立するかの判断は難しいところがあるが、発振を抑えるという考えからも$T_i\fallingdotseq4\tau_c$付近が扱いやすいことがわかる。

### (1)$D>=0$区間

Dが正の値であるときは、前出の式にそのまま代入することがかのうである。$D=0$のとき

```math
\begin{align*}
\lim_{D\rightarrow0}\cosh{\left(\frac{\sqrt{D}}{2}t\right)}&=1\\
\lim_{D\rightarrow0}\frac{2}{t\sqrt{D}}\sinh{\left(\frac{\sqrt{D}}{2}t\right)}&=1
\end{align*}
```

を用いて解くことができる。
(4-13)式のこの区間では、

```math
\begin{align*}
\cosh{\left(\frac{\sqrt{D}}{2}t\right)}+\frac{1-AK_p}{\tau\sqrt{D}}\sinh{\left(\frac{\sqrt{D}}{2}t\right)}\\
1+\frac{1}{2}(1-AK_p)t \\
\end{align*}
```

は両方とも$t>0$の区間で増加関数であるため、発振しないことがわかる。

### (2) $D<0$ 区間
$D<0$の時にとくには、以下の性質を使う。

```math
\begin{align*}
\cosh(x)&=\cos(jx)\\
\sinh(x)&=-j\sin(jx)\\
j\sqrt{D}&=\sqrt{-D}
\end{align*}
```

# 参考文献
1. http://www.isc.meiji.ac.jp/~mcelab/kikai_jikken1/20141210_jikken1_ppt.pdf
