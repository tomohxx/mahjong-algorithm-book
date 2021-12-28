# 和了確率の表式

連立確率漸化式(1)で表された和了確率の解$p^{(d)}_t$を求める.

$$
\left\{
\begin{aligned}
p^{(0)}_{t+1} &= \left( 1- \frac{a_0}{S-t} \right) p^{(0)}_t & p^{(0)}_0 &= 1 \\
p^{(i)}_{t+1} &= \left( 1- \frac{a_i}{S-t} \right) p^{(i)}_t + \frac{a_{i-1}}{S-t} p^{(i-1)}_t & p^{(i)}_0 &= 0 & (0 \le i < d) \\
p^{(d)}_{t+1} &= \frac{a_{d-1}}{S-t} p^{(d-1)}_t + p^{(d)}_{t} & p^{(d)}_0 &= 0
\end{aligned}
\right.
\tag{1}
$$

(1)式を行列で表す. なお以降$a_d = 0$とする.

$$
A_t =
\begin{pmatrix}
1-\frac{a_0}{S-t} & 0 & \dots & 0 & 0 \\
\frac{a_0}{S-t} & 1-\frac{a_1}{S-t} & \dots & 0 & 0 \\
\vdots & \vdots & \ddots & \vdots & \vdots \\
0 & 0 & \dots & 1-\frac{a_{d-1}}{S-t} & 0 \\
0 & 0 & \dots & \frac{a_{d-1}}{S-t} & 1-\frac{a_d}{S-t} \\
\end{pmatrix}
\tag{2}
$$

$$
P_t = 
\begin{pmatrix}
p^{(0)}_t \\
p^{(1)}_t \\
\vdots \\
p^{(d-1)}_t \\
p^{(d)}_t
\end{pmatrix}
\tag{3}
$$

とすると

$$
P_{t+1} = A_t P_t
\tag{4}
$$

となる. (4)式を解くと

$$
P_t = \left\{ \prod_{u=0}^{t-1} A_u \right\} P_0
\tag{5}
$$

となる. ここで$A_t$は下三角行列であるから$\prod_{u=0}^{t-1} A_u$も下三角行列であり, その対角成分は$A_u \ (0 \le u \le t-1)$の対角成分の積である. また下三角行列の固有値は対角成分であるが$a_i \ (0 \le i \le d-1)$はすべて異なるから$\prod_{u=0}^{t-1} A_u$は対角化可能である. よって適当な対角行列$D$と正則行列$X$が存在して

$$
\prod_{u=0}^{t-1} A_u = X D X^{-1}
\tag{6}
$$

のように書ける. (6)式より適当な係数$c_i \ (0 \le i \le d)$を定めれば$p^{(d)}_t$は

$$
\begin{aligned}
p^{(d)}_t &= \sum_{i=0}^{d} c_i \prod_{u=0}^{t-1} \left( 1 - \frac{a_i}{S-t} \right) \\
&= \sum_{i=0}^{d} c_i \times \frac{_{S-a_i}P_t}{_{S}P_t} \\
&= \frac{1}{_{S}P_t} \sum_{i=0}^{d} c_i \times _{S-a_i} P_t
\end{aligned}
\tag{7}
$$

と表せる. この係数$c_i$を初期条件

$$
\begin{aligned}
p^{(d)}_0 &= p^{(d)}_1 = \dots = p^{(d)}_{d-1} = 0 \\
p^{(d)}_{d} &= \prod_{i=0}^{d-1} \frac{a_i}{S-i} = \frac{1}{_SP_d} \prod_{i=0}^{d-1} a_i = \frac{C}{_SP_d}
\end{aligned}
\tag{8}
$$

によって定める. これは

$$
\begin{pmatrix}
1 & 1 & \dots & 1 & 1 \\
S-a_0 & S-a_1 & \dots & S-a_{d-1} & S-a_d \\
\vdots & \vdots & \ddots & \vdots & \vdots \\
_{S-a_0}P_{d-1} & _{S-a_1}P_{d-1} & \dots & _{S-a_{d-1}}P_{d-1} & _{S-a_{d}}P_{d-1} \\
_{S-a_0}P_{d} & _{S-a_1}P_{d} & \dots & _{S-a_{d-1}}P_{d} & _{S-a_{d}}P_{d} \\
\end{pmatrix}
\begin{pmatrix}
c_0 \\
c_1 \\
\vdots \\
c_{d-1} \\
c_{d}
\end{pmatrix}
=
\begin{pmatrix}
0 \\
0 \\
\vdots \\
0 \\
C
\end{pmatrix}
\tag{9}
$$

と表せる. (9)式の行列を$V$と表すことにする. $\det V$はヴァンデルモンド行列式のように計算できて

$$
\det{V}  = \prod_{0 \le j < k \le d} (a_j - a_k)
\tag{10}
$$

である. これとクラメルの公式を使って係数$c_i$は

$$
c_i = \frac{\det{V_i}}{\det{V}}
\tag{11}
$$

と表せる. $\det V_i$は余因子展開を使えば

$$
\det V_i = {(-1)}^{d+i} C \prod_{0 \le j < k \le d,\  j \neq i,\ k \neq i} (a_j - a_k)
\tag{12}
$$

と書ける. よって(11)式は

$$
c_i = C \prod_{j \neq i} \frac{1}{a_j - a_i}
\tag{13}
$$

となる. ここで(7)式を整理する($a_d = 0$に注意).

$$
\begin{aligned}
p^{(d)}_t &= \sum_{i=0}^{d-1} c_i \times \frac{_{S-a_i}P_t}{_SP_t} + c_n \\
&= \sum_{i=0}^{d-1} c_i \times \frac{_{S-a_i}P_t}{_SP_t} + \sum_{i=0}^{d-1} c_i \\
&= \sum_{i=0}^{d-1} c_i \times \left( \frac{_{S-a_i}P_t}{_SP_t} - 1 \right)
\end{aligned}
\tag{14}
$$

また

$$
\begin{aligned}
c_i &= C \left\{ \prod_{0 \le j \le d-1,\ j \neq i} \frac{1}{a_j - a_i} \right\} \times \frac{1}{a_d - a_i} \\
&= -C \left\{ \prod_{0 \le j \le d-1,\ j \neq i} \frac{1}{a_j - a_i} \right\} \times \frac{1}{a_i} & (0 \le i \le d-1)
\end{aligned}
\tag{15}
$$

なので, これを(14)式に代入して整理すると

$$
\begin{aligned}
p^{(d)}_t &= C \sum_{i=0}^{d-1} \frac{1}{a_i} \left( 1 - \frac{_{S-a_i}P_t}{_SP_t} \right) \prod_{j \neq i} \frac{1}{a_j - a_i} \\
&= \left\{ \prod_{i=0}^{d-1} a_i \right\} \times \sum_{i=0}^{d-1} \frac{1}{a_i} \left( 1 - \frac{_{S-a_i}P_t}{_SP_t} \right) \prod_{j \neq i} \frac{1}{a_j - a_i}
\end{aligned}
\tag{16}
$$

となり解を得る.


