# 和了と聴牌

## 定義

通常の意味での和了および聴牌を拡張して, $3N$和了と$3N+2$和了, $3N+2$聴牌と$3N+1$聴牌を定義する.

**$3N$和了**
: $h \in H$が$N$ ($N$は任意の自然数)組の面子に分解できるならば, $h$は$3N$和了である.

**$3N+2$和了**
: $h \in H$が$N$ ($N$は任意の自然数)組の面子と 1 組の雀頭に分解できるならば, $h$は$3N+2$和了である.

**$3N+2$聴牌**
: $h \in H$に 1 枚牌を加えることで$3N$和了になるならば, $h \in H$は$3N+1$聴牌である. また, このときに加えることができる牌の集合を待ちという.

**$3N+1$聴牌**
: $h \in H$に 1 枚牌を加えることで$3N+2$和了になるならば, $h \in H$は$3N+1$聴牌である. また, このときに加えることができる牌の集合を待ちという.

<figure text-align="center">
  <img src="../img/states.svg"/>
  <figcaption>図1: 手牌の状態</figcaption>
</figure>

## 和了(聴牌)になるための必要条件

一色手の牌の添字の和$s = \sum_{i=0}^{8} i h^n_i$について考える. $s$を用いることで和了(聴牌)になるための必要条件を得られる. なお, 以下の合同式では$\bmod 3$を省略している.

**3N 和了になるための必要条件(定理 1)**
: $s \equiv 0$が成り立つ.

> 同一の 3 数の和および連続する 3 数の和はともに 3 で割り切れるため成り立つ.

**3N+2 和了になるための必要条件(定理 2)**
: $p \equiv 2s$なる$p$番目の牌が雀頭となる.

> $s \equiv 2p$より$2s \equiv 4p \equiv p$.

**3N+2 聴牌になるための必要条件(定理 3)**
: $x \equiv 2s$なる$x$番目の牌が待ちに含まれる.

> 定理 1 より$s+x \equiv 0$. 整理して$x \equiv -s \equiv 2s$.

**3N+1 聴牌になるための必要条件(定理 4)**
: $x + p \equiv 2s$なる$x, p$について$x$番目の牌が待ちに含まれ$p$番目の牌が雀頭となる.

> 定理 2 より$p \equiv 2(s+x)$. 整理して$2s \equiv p - 2x \equiv p + x$.

## 一色手の和了(聴牌)判定

### 3N 和了判定

$h^n_i$の要素を$i=0$から走査し, 面子を取り出していく. $h^n_i = 1, 2, 4$のときは$h^n_i \bmod 3$個の順子を取り出す. $h^n_i = 3$のときは, 可能性として順子 3 個を取り出すか刻子 1 個を取り出すかの 2 通りの操作が考えられるが, 前者が可能である場合は後者も可能となる一方で, 後者が可能である場合はいつも前者が可能となるわけではない. よって$h^n_i = 3$のときは刻子を取り出すと決めてしまってよい.

```cpp
bool iswh0(const int* h)
{
  int a = h[0], b = h[1];

  for (int i = 0; i < 7; ++i) {
    if (int r = a % 3; b >= r && h[i + 2] >= r) {
      a = b - r;
      b = h[i + 2] - r;
    }
    else return false;
  }
  return a % 3 == 0 && b % 3 == 0;
}
```

### 3N+2 和了判定

雀頭候補を取り出した手牌が$3N$和了であるか判定する.

```cpp
bool iswh2(int* h)
{
  int s = 0;

  for (int i = 0; i < 9; ++i) {
    s += i * h[i];
  }

  for (int p = s * 2 % 3; p < 9; p += 3) {
    if (h[p] >= 2) {
      h[p] -= 2;

      if (iswh0(h)) {
        h[p] += 2;
        return true;
      }
      else h[p] += 2;
    }
  }
  return false;
}
```

### 3N+2 聴牌判定

```cpp
int isrh2(int* h)
{
  int s = 0, wait = 0;

  for (int i = 0; i < 9; ++i) {
    s += i * h[i];
  }

  for (int x = s * 2 % 3; x < 9; x += 3) {
    if (h[x] < 4) {
      ++h[x];

      if (iswh0(h)) {
        wait ^= 1 << x;
      }
      --h[x];
    }
  }
  return wait;
}
```

### 3N+1 聴牌判定

```cpp
int isrh1(int* h)
{
  int s = 0, wait = 0;

  for (int i = 0; i < 9; ++i) {
    s += i * h[i];
  }

  for (int x = 0; x < 9; ++x) {
    if (h[x] < 4) {
      ++h[x];

      for (int p = (s * 2 - x) % 3; p < 9; p += 3) {
        if (h[p] >= 2) {
          h[p] -= 2;

          if (iswh0(h)) {
            h[p] += 2;
            wait ^= 1 << x;
            break;
          }
          else h[p] += 2;
        }
      }
      --h[x];
    }
  }
  return wait;
}
```

## 一色手の面子分解パターン列挙

和了手牌に対して面子分解パターンを列挙する. 大抵の場合, 面子分解パターンは 1 つで上記の和了判定と同時に面子分解パターンを取得できる. しかし一部の和了手牌は複数の面子分解パターンをもつため, それらを列挙するためには次の 2 つの不定性に注意する必要がある.

1. 雀頭の不定性
1. 刻子の不定性

1 つ目の雀頭の不定性とは, 例えば手牌 11223344 が[11][234][234]と[123][123][44]のように複数のパターンで分解され雀頭が 1 通りに定まらないことである. 2 つ目の刻子の不定性とは, 例えば手牌 111222333 が[111][222][333]と[123][123][123]のように複数のパターンで分解され刻子と順子のどちらを取り出すか定まらないことである. これらの不定性は単独で存在することもあれば同時に存在することもある. 以上を考慮すると一色手の面子分解パターンを列挙するアルゴリズムは次のようになる.

コマンドライン引数に手牌を表す文字列を与えると面子分解パターンを列挙する.

```cpp
#include <algorithm>
#include <functional>
#include <iostream>

struct Mentsu {
  int jantou = 0;  // 雀頭の数 (0 to 1)
  int koutsu = 0;  // 刻子の数 (0 to 1)
  int shuntsu = 0; // 順子の数 (0 to 4)
};

bool iswh0(const int* h, Mentsu* m)
{
  int a = h[0], b = h[1];

  for (int i = 0; i < 7; ++i) {
    if (a >= 3) {
      a -= 3;
      m[i].koutsu = 1;
    }

    if (const int r = a; b >= r && h[i + 2] >= r) {
      a = b - r;
      b = h[i + 2] - r;
      m[i].shuntsu += r;
    }
    else return false;
  }

  if (a == 3) {
    a = 0;
    m[7].koutsu = 1;
  }

  if (b == 3) {
    b = 0;
    m[8].koutsu = 1;
  }

  return a == 0 && b == 0;
}

void iswh2(int* h, Mentsu* m, std::function<void(Mentsu*)> func)
{
  int s = 0;

  for (int i = 0; i < 9; ++i) {
    s += i * h[i];
  }

  for (int p = s * 2 % 3; p < 9; p += 3) {
    if (h[p] >= 2) {
      h[p] -= 2;
      std::fill(m, m + 9, Mentsu{});

      if (iswh0(h, m)) {
        h[p] += 2;
        m[p].jantou = 1;
        func(m);
      }
      else h[p] += 2;
    }
  }
}

void koutsu3(Mentsu* m, std::function<void(Mentsu*)> func)
{
  for (int i = 0; i < 7; ++i) {
    if (m[i].koutsu && m[i + 1].koutsu && m[i + 2].koutsu) {
      m[i].koutsu = 0;
      m[i + 1].koutsu = 0;
      m[i + 2].koutsu = 0;
      m[i].shuntsu += 3;
      func(m);
      m[i].koutsu = 1;
      m[i + 1].koutsu = 1;
      m[i + 2].koutsu = 1;
      m[i].shuntsu -= 3;
    }
  }
}

void print(const Mentsu* m)
{
  for (int i = 0; i < 9; ++i) {
    if (m[i].jantou) {
      std::cout << "[" << i + 1 << "," << i + 1 << "],";
    }
    else if (m[i].koutsu) {
      std::cout << "[" << i + 1 << "," << i + 1 << "," << i + 1 << "],";
    }
    else if (m[i].shuntsu) {
      for (int j = 0; j < m[i].shuntsu; ++j) {
        std::cout << "[" << i + 1 << "," << i + 2 << "," << i + 3 << "],";
      }
    }
  }

  std::cout << "\n";
}

int main(int argc, char* argv[])
{
  if (argc != 2) {
    std::cout << argc << std::endl;
    return 1;
  }

  int h[9] = {};
  Mentsu m[9] = {};

  for (int i = 0; i < 14; ++i) {
    if (argv[1][i] < '1' && argv[1][i] > '9') {
      return 1;
    }
    ++h[argv[1][i] - '1'];
  }

  iswh2(h, m, [](Mentsu* m) {
    print(m);
    koutsu3(m, [](Mentsu* m) { print(m); });
  });

  return 0;
}
```

入力例 1:

```
./a.out 12344455888999
```

出力例 1:

```
[1,2,3],[4,4,4],[5,5],[8,8,8],[9,9,9],
```

入力例 2:

```
./a.out 22334455667788
```

出力例 2:

```
[2,2],[3,4,5],[3,4,5],[6,7,8],[6,7,8],
[2,3,4],[2,3,4],[5,5],[6,7,8],[6,7,8],
[2,3,4],[2,3,4],[5,6,7],[5,6,7],[8,8],
```

入力例 3:

```
./a.out 11122233344455
```

出力例 3:

```
[1,1,1],[2,2],[3,4,5],[3,4,5],
[1,1,1],[2,2,2],[3,3,3],[4,4,4],[5,5],
[1,2,3],[1,2,3],[1,2,3],[4,4,4],[5,5],
[1,1,1],[2,3,4],[2,3,4],[2,3,4],[5,5],
```
