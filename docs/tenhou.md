# 天和の確率

## 概要

天和の確率は以下の式で与えられる.

$$
\mbox{天和の確率} = \frac{\mbox{四面子一雀頭の数①}-\mbox{二盃口の数②}+\mbox{七対子の数③}+\mbox{国士無双の数④}}{\mbox{全手牌の数⑤}}
$$

③, ④, ⑤は簡単な数式で表せる.

$$
\begin{aligned}
\mbox{七対子の数③} &= {}_{34}C_{7} \times ({}_4C_2)^{7} \\
\mbox{国士無双の数④} &= 13 \times {}_4C_2 \times ({}_4C_1)^{12} \\
\mbox{全手牌の数⑤} &= {}_{136}C_{14}
\end{aligned}
$$

①と②は簡単な数式で表せないが, [5. 連立集合漸化式](ssrf.md#_3)を用いて高速に計算することができる.

## ソースコード

```cpp
#include <iostream>
#include <functional>
#include <vector>

// 手牌生成(全幅)
template <int N>
void generate(int n, int m, std::function<void(std::vector<int>&)> func)
{
  static std::vector<int> h(N);

  if (n == N) {
    func(h);
  }
  else {
    for (int i = std::max(0, m - 4 * (N - 1 - n)); i <= std::min(4, m); ++i) {
      h[n] = i;
      generate<N>(n + 1, m - i, func);
    }
  }
}

// 3N和了判定
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

// 3N+2和了判定
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

// 七対子判定
bool issp(const int* h)
{
  for (int i = 0; i < 9; ++i) {
    if (h[i] != 0 && h[i] != 2) return false;
  }
  return true;
}

// 手牌のパターン数
template <class Iter>
int patterns(Iter begin, Iter end)
{
  // 4Cr
  static std::vector<int> combin = {1, 4, 6, 4, 1};

  int ret = 1;

  for (; begin != end; ++begin) {
    ret *= combin[*begin];
  }
  return ret;
}

// べき乗(x^n)
long long pow(const int x, const int n)
{
  if (n == 0) return 1;
  long long ret = pow(x * x, n / 2);
  if (n & 1) ret = ret * x;
  return ret;
}

// 組み合わせ(nCr)
__int128_t combin(const int n, const int r)
{
  if (r == 0) return 1;
  else return (n - r + 1) * combin(n, r - 1) / r;
}

// パターン数を管理するクラス
struct States {
  // 四面子一雀頭のパターン数
  long long first = 0;
  // 二盃口のパターン数
  long long second = 0;

  States& operator+=(const States& rhs)
  {
    first += rhs.first;
    second += rhs.second;
    return *this;
  }

  States& operator*=(const States& rhs)
  {
    first *= rhs.first;
    second *= rhs.second;
    return *this;
  }
};

const States operator+(const States& lhs, const States& rhs)
{
  return States(lhs) += rhs;
}

const States operator*(const States& lhs, const States& rhs)
{
  return States(lhs) *= rhs;
}

int main()
{
  // |A^n_m|
  std::vector<std::vector<States>> a(4, std::vector<States>(5));
  // |B^n_m|
  std::vector<std::vector<States>> b(4, std::vector<States>(5));

  for (int m = 0; m <= 4; ++m) {
    // 数牌3N和了のパターン数と二盃口のパターン数を数える
    generate<9>(0, 3 * m, [&](std::vector<int>& h) {
      if (iswh0(h.data())) {
        auto num = patterns(h.begin(), h.end());

        a[0][m].first += num;
        if (issp(h.data())) a[0][m].second += num;
      }
    });

    // 数牌3N+2和了のパターン数と二盃口のパターン数を数える
    generate<9>(0, 3 * m + 2, [&](std::vector<int>& h) {
      if (iswh2(h.data())) {
        auto num = patterns(h.begin(), h.end());

        b[0][m].first += num;
        if (issp(h.data())) b[0][m].second += num;
      }
    });

    // 字牌3N和了のパターン数を数える
    generate<7>(0, 3 * m, [&](std::vector<int>& h) {
      std::vector<int> cnt(5, 0);

      for (int i = 0; i < 7; ++i) {
        ++cnt[h[i]];
      }

      if (cnt[1] == 0 && cnt[2] == 0 && cnt[4] == 0) {
        a[3][m].first += patterns(h.begin(), h.end());
      }
    });

    // 字牌3N+2和了のパターン数を数える
    generate<7>(0, 3 * m + 2, [&](std::vector<int>& h) {
      std::vector<int> cnt(5, 0);

      for (int i = 0; i < 7; ++i) {
        ++cnt[h[i]];
      }

      if (cnt[1] == 0 && cnt[2] == 1 && cnt[4] == 0) {
        b[3][m].first += patterns(h.begin(), h.end());
      }
    });
  }

  for (int m = 1; m < 3; ++m) {
    std::copy(a[0].begin(), a[0].end(), a[m].begin());
    std::copy(b[0].begin(), b[0].end(), b[m].begin());
  }

  a[3][0].second = 1;
  b[3][0].second = 42; // 4C2*7

  // DPテーブルA
  std::vector<States> dp_a(a[0]);
  // DPテーブルB
  std::vector<States> dp_b(b[0]);

  // DPテーブルを更新する
  for (int n = 0; n < 3; ++n) {
    for (int m = 4; m >= 0; --m) {
      States tmp_a, tmp_b;

      for (int l = 0; l <= m; ++l) {
        tmp_a += dp_a[l] * a[n + 1][m - l];
        tmp_b += dp_b[l] * a[n + 1][m - l] + dp_a[l] * b[n + 1][m - l];
      }
      dp_a[m] = tmp_a;
      dp_b[m] = tmp_b;
    }
  }

  // 七対子のパターン数
  long long num_sp = combin(34, 7) * pow(combin(4, 2), 7);
  // 国士無双のパターン数
  long long num_to = 13 * combin(4, 2) * pow(combin(4, 1), 12);
  // 全手牌のパターン数
  long long num_total = combin(136, 14);

  // 分子
  auto numelator = dp_b[4].first - dp_b[4].second + num_sp + num_to;

  std::cout.precision(15);
  std::cout << numelator << " / " << num_total << " = " << static_cast<double>(numelator) / num_total << std::endl;

  return 0;
}
```

出力:

```
12859078207674 / 4250305029168216000 = 3.02544831945638e-06
```
