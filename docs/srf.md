# 集合漸化式

## 定義

$H^n$の要素のうち, $m$面子に分解できるものの集合を$A^n_m$, $m$面子一雀頭に分解できるものの集合を$B^n_m$とする.

| 部分集合 | 説明                                                   |
| :------- | :----------------------------------------------------- |
| $A^n_m$  | $H^n$の要素のうち, $m$面子に分解できるものの集合       |
| $B^n_m$  | $H^n$の要素のうち, $m$面子一雀頭に分解できるものの集合 |

すべての$n \in [0, 3], m \in [0, 4]$について$A^n_m, B^n_m$が与えらたとき, 以下の**集合漸化式**を解くことですべての$3N+2$和了形を得られる. ここで$m$面子一雀頭からなる$3N+2$和了形の集合は$B^{(3)}_m$である.

$$
\left\{
\begin{aligned}
A^{(n+1)}_m &= \bigcup^{m}_{l=0} \left\{ A^{(n)}_l \times A^{n+1}_{m-l} \right\} & A^{(0)}_m &= A^0_m \\
B^{(n+1)}_m &= \bigcup^{m}_{l=0} \left\{ B^{(n)}_l \times A^{n+1}_{m-l} \cup A^{(n)}_l \times B^{n+1}_{m-l} \right\} & B^{(0)}_m &= B^0_m
\end{aligned}
\right. \\
$$

集合漸化式は数どうしの演算についての通常の漸化式と異なり, 集合どうしの演算についての漸化式である. これを解こうとすると集合をどのようなデータ構造で表現するかなどの疑問が生じるが, 実際にこれを解くことはまずなく大抵は通常の漸化式に帰着させる. なお, 集合漸化式から導かれたアルゴリズムは動的計画法の考え方を用いているといえる.

## 和了形の数

集合漸化式の最も簡単な応用例は和了形の数を得ることである. $3N+2$和了形の数$|B^{(3)}_m|$は以下の連立漸化式の解として得られる. これは[天和の確率](tenhou.md)を計算することに利用できる.

$$
\left\{
\begin{aligned}
|A^{(n+1)}_m| &= \sum^{m}_{l=0} \left\{ |A^{(n)}_l| \times |A^{n+1}_{m-l}| \right\} & |A^{(0)}_m| &= |A^0_m| \\
|B^{(n+1)}_m| &= \sum^{m}_{l=0} \left\{ |B^{(n)}_l| \times |A^{n+1}_{m-l}| + |A^{(n)}_l| \times |B^{n+1}_{m-l}| \right\} & |B^{(0)}_m| &= |B^0_m|
\end{aligned}
\right. \\
$$

## 向聴数

ここでは向聴数を「聴牌するために必要な牌の枚数の最小値」と定義する. 聴牌を基準とした向聴数は計算上不便なので, 「和了するために必要な牌の枚数の最小値」として置換数を導入する. 向聴数$S(h)$と置換数$T(h)$の関係は次式で表される.

$$
S(h) = T(h) - 1
$$

$h^n$から$g^n$に変化させるために必要な牌の枚数を距離$d(g^n, h^n)$として以下のように定義する.

$$
d(g^n, h^n) = \frac{1}{2} \sum_{i} (|g^n_i - h^n_i| + g^n_i - h^n_i)
$$

これを用いて手牌$h$を手牌$g$に変化させるのに必要な牌の枚数は

$$
\sum_{n=0}^{3} d(g^n, h^n)
$$

と表せる. 距離を用いて手牌$h$の置換数を以下のように定義する.

$$
T(h) = \min_{b \in B^{(3)}_m} \sum_{n=0}^{3} d(b^n, h^n)
$$

ここで, 事前に以下の値がわかっているとする.

$$
\left\{
\begin{aligned}
u^n_m &= \min_{a \in A^n_m} d(a^n, h^n) \\
t^n_m &= \min_{b \in B^n_m} d(b^n, h^n) \\
\end{aligned}
\right.
$$

このとき手牌$h$の置換数$T(h)$は以下の連立漸化式の解$t^{(3)}_m$に等しい.

$$
\left\{
\begin{aligned}
u^{(n+1)}_m &= \min_{0 \le l \le m} \left\{ u^{(n)}_l + u^{n+1}_{m-l} \right\} & u^{(0)}_m &= u^0_m \\
t^{(n+1)}_m &= \min_{0 \le l \le m} \left\{ \min\{ t^{(n)}_l + u^{n+1}_{m-l} , u^{(n)}_l + t^{n+1}_{m-l} \} \right\} & t^{(0)}_m &= t^0_m
\end{aligned}
\right.
$$

このように前処理を行うことで置換数(向聴数)を高速に計算できる. ソースコードは[shanten-number](https://github.com/tomohxx/shanten-number)を参照すること.

## 配牌時向聴数

配牌時の向聴数については[配牌時向聴数](shanten.md)を参照すること.

## 有効牌/不要牌

置換数(向聴数)と合わせて有効牌/不要牌を同時に計算できる. ソースコードは[necessary-or-unnecessary-tiles](https://github.com/tomohxx/necessary-or-unnecessary-tiles)を参照すること.

## 聴牌形

和了形と同様に聴牌形も集合漸化式で表せる. 前述の$A^n_m, B^n_m$に加えて以下の集合$C^n_m, D^n_m$を考える.

| 部分集合 | 説明                                                                      |
| :------- | :------------------------------------------------------------------------ |
| $A^n_m$  | $H^n$の要素のうち, $m$面子に分解できるものの集合                          |
| $B^n_m$  | $H^n$の要素のうち, $m$面子一雀頭に分解できるものの集合                    |
| $C^n_m$  | $H^n$の要素のうち, あと 1 枚追加すると$m$面子に分解できるものの集合       |
| $D^n_m$  | $H^n$の要素のうち, あと 1 枚追加すると$m$面子一雀頭に分解できるものの集合 |

すべての$n \in [0, 3], m \in [0, 4]$について$A^n_m, B^n_m, C^n_m, D^n_m$が与えらたとき, 以下の集合漸化式を解くことですべての$3N+1$聴牌形の集合$D^{(3)}_m$を得られる.

$$
\left\{
\begin{aligned}
A^{(n+1)}_m &= \bigcup^{m}_{l=0} \left\{ A^{(n)}_l \times A^{n+1}_{m-l} \right\} & A^{(0)}_m &= A^0_m \\
B^{(n+1)}_m &= \bigcup^{m}_{l=0} \left\{ B^{(n)}_l \times A^{n+1}_{m-l} \cup A^{(n)}_l \times B^{n+1}_{m-l} \right\} & B^{(0)}_m &= B^0_m \\
C^{(n+1)}_m &= \bigcup^{m}_{l=0} \left\{ C^{(n)}_l \times A^{n+1}_{m-l} \cup A^{(n)}_l \times C^{n+1}_{m-l} \right\} & C^{(0)}_m &= C^0_m \\
D^{(n+1)}_m &= \bigcup^{m}_{l=0} \left\{ D^{(n)}_l \times A^{n+1}_{m-l} \cup A^{(n)}_l \times D^{n+1}_{m-l} \cup B^{(n)}_l \times C^{n+1}_{m-l-1} \cup C^{(n)}_l \times B^{n+1}_{m-l-1} \right\} & D^{(0)}_m &= D^0_m
\end{aligned}
\right.
$$

和了形の数と同様に聴牌形の数も計算することができる. 特に, 集合$C^n_m, D^n_m$のうちある牌を待つ集合だけを取り出せば, その牌を待つ聴牌形の集合を得られる. この集合の要素数をすべての聴牌形の集合の要素数で割ることで, すべての聴牌形の出現確率がそれぞれ等しいとしたときにある牌を待つ確率を計算できる. つまり, 最も単純な仮定の下で牌の危険度を予測できる. これについてソースコードは[waits-predictor](https://github.com/tomohxx/waits-predictor)を参照すること.

$$
(\text{ある牌を待つ確率}) = \frac{(\text{その牌を待つ聴牌形の数})}{(\text{すべての聴牌形の数})}
$$
