# 配牌時有効牌の枚数

## 概要

配牌時(手牌の枚数が13枚のとき)の有効牌の枚数の平均値を計算する. 有効牌の枚数は各色の数牌に対する反転操作(例えば1mを9mに, 2pを8pに移す)と字牌どうしの入れ替えに関して不変である, という性質を利用して計算量を小さくできる.

## ソースコード

以下のソースコードを実行する場合は, [necessary-or-unnecessary-tiles](https://github.com/tomohxx/necessary-or-unnecessary-tiles)から`calsht_dw.hpp`をインクルードしインデックスファイル(`index_dw_s.txt`と`index_dw_h.txt`)を実行ファイルと同じディレクトリに配置すること. また, 実行に数分間かかるため注意すること.

```cpp
#include <chrono>
#include <fstream>
#include <functional>
#include <iostream>
#include <unordered_map>
#include <numeric>
#include <valarray>
#include <vector>
#include <boost/container_hash/hash.hpp>
#include "calsht_dw.hpp"
using Vec = std::vector<int>;
using Iter = std::vector<Vec>::iterator;
struct Key;
struct Hash;
using Keys = std::unordered_map<Key, int, Hash>;

struct Key {
  Vec hand;
  long long dens;
};

template <bool IsHonor>
Key make_key(const Vec& v, const long long d);

template <>
Key make_key<true>(const Vec& v, const long long d)
{
  Vec tmp(v);
  std::sort(tmp.begin(), tmp.end());
  return Key{tmp, d};
}

template <>
Key make_key<false>(const Vec& v, const long long d)
{
  Vec tmp(v);
  std::reverse(tmp.begin(), tmp.end());
  return Key{min(tmp, v), d};
}

struct Hash {
  std::size_t operator()(const Key& key) const
  {
    return boost::hash_range(key.hand.begin(), key.hand.end());
  }
};

bool operator==(const Key& lhs, const Key& rhs)
{
  return lhs.hand == rhs.hand;
}

template <bool IsHonor>
void generate(int n, int m, Keys& keys)
{
  static const Vec c = {1, 4, 6, 4, 1};
  static constexpr int N = IsHonor ? 7 : 9;
  static Vec hand(N);
  static Vec dens(N + 1, 1);

  if (n == N) {
    const auto key = make_key<IsHonor>(hand, dens[n]);

    if (auto itr = keys.find(key); itr == keys.end()) {
      keys.emplace(key, 1);
    }
    else {
      ++itr->second;
    }
  }
  else {
    for (int i = std::max(0, m - 4 * (N - 1 - n)); i <= std::min(4, m); ++i) {
      hand[n] = i;
      dens[n + 1] = dens[n] * c[i];
      generate<IsHonor>(n + 1, m - i, keys);
    }
  }
}

inline int coef(const int i, const int j, const int k)
{
  if (i == j && j == k) {
    return 1;
  }
  else if (i == j || j == k || k == i) {
    return 3;
  }
  else {
    return 6;
  }
}

int count(const Vec& hand, const int64_t wait)
{
  int tmp = 0;

  for (int i = 0; i < K; ++i) {
    if ((wait >> i) & 1) {
      tmp += 4 - hand[i];
    }
  }
  return tmp;
}

int main()
{
  constexpr int M = 13;

  CalshtDW calsht;

  calsht.initialize("./");

  const auto start = std::chrono::system_clock::now();

  std::vector<Keys> keys1(M + 1);
  std::vector<Keys> keys2(M + 1);

  for (int m = 0; m <= M; ++m) {
    generate<false>(0, m, keys1[m]);
    generate<true>(0, m, keys2[m]);
  }

  std::valarray<long long> cnt_sht(8);
  std::valarray<__int128_t> cnt_wait(8);

  for (int i = 0; i <= M; ++i) {
    for (int j = i; j <= M - i; ++j) {
      for (int k = j; k <= M - i - j; ++k) {
        std::valarray<long long> tmp_sht(8);
        std::valarray<__int128_t> tmp_wait(8);

        for (const auto& [key0, value0] : keys1[i]) {
          for (const auto& [key1, value1] : keys1[j]) {
            for (const auto& [key2, value2] : keys1[k]) {
              for (const auto& [key3, value3] : keys2[M - i - j - k]) {
                Vec hand(K);

                std::copy(key0.hand.begin(), key0.hand.end(), hand.begin());
                std::copy(key1.hand.begin(), key1.hand.end(), hand.begin() + 9);
                std::copy(key2.hand.begin(), key2.hand.end(), hand.begin() + 18);
                std::copy(key3.hand.begin(), key3.hand.end(), hand.begin() + 27);

                const auto [sht, mode, disc, wait] = calsht(hand, M / 3, 7);
                const auto dens = key0.dens * value0 * key1.dens * value1 * key2.dens * value2 * key3.dens * value3;

                tmp_sht[sht] += dens;
                tmp_wait[sht] += count(hand, wait) * dens;
              }
            }
          }
        }

        cnt_sht += tmp_sht * coef(i, j, k);
        cnt_wait += tmp_wait * coef(i, j, k);
      }
    }
  }

  const auto end = std::chrono::system_clock::now();
  const auto total = cnt_sht.sum();
  double ev = 0.0;

  std::cout << "Shanten\tCount\tProp\tAvg Wait" << std::endl;

  for (int i = 0; i < 8; ++i) {
    ev += (i - 1) * cnt_sht[i];
    std::cout << i - 1 << "\t" << cnt_sht[i] << "\t"
              << static_cast<double>(cnt_sht[i]) / total << "\t"
              << static_cast<double>(cnt_wait[i]) / cnt_sht[i] << std::endl;
  }

  std::cout << "Number of Tiles: " << M << std::endl;
  std::cout << "Total: " << total << std::endl;
  std::cout << "Time (msec.): " << std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count() << std::endl;
  std::cout << "Avg Shanten: " << ev / total << std::endl;
  std::cout << "Avg Wait: " << static_cast<double>(cnt_wait.sum()) / total << std::endl;

  return 0;
}
```

出力:

```
Shanten Count   Prop    Avg Wait
-1      0       0       -nan
0       39270395383132  8.1175e-05      4.5843
1       3006175115638776        0.006214        12.2671
2       45249205945148216       0.0935337       23.3215
3       175141291509958900      0.362031        40.4909
4       192909046305573888      0.398758        60.6136
5       63384201353756672       0.13102 77.8174
6       4045365540028416        0.00836209      100.154
Number of Tiles: 13
Total: 483774556165488000
Time (msec.): 212463
Avg Shanten: 3.57968
Avg Wait: 52.1202
```

配牌時(手牌の枚数が13枚のとき)有効牌の枚数の平均値は`52.1202`である.
