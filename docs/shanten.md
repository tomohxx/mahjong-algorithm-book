# 配牌時向聴数

## 概要

すべての手牌について向聴数を計算しようとすると計算量が大きくなってしまう. 事前に数牌と字牌の向聴数を計算するためのパラメータについて集約し, 同じパラメータがいくつ存在するかをメモしておく. 手牌の枚数が予め決められた数になるようなすべてのパラメータの組み合わせを全探索する. このとき向聴数を計算しながら同じ向聴数となる手牌の数も計算することで計算量を小さくできる. また, 数牌の色どうしを入れ替えても向聴数は変わらない事実を利用することでさらに計算量を小さくできる.

## ソースコード

以下のソースコードを実行する場合は, [shanten-number](https://github.com/tomohxx/shanten-number)からインデックスファイル(`index_s.txt`と`index_h.txt`)を入手し実行ファイルと同じディレクトリに配置すること.

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
using Vec = std::vector<int>;
using Iter = std::vector<Vec>::iterator;
struct Key;
struct Hash;
using Keys = std::unordered_map<Key, int, Hash>;

struct Key{
  // 距離テーブルへのイテレータ
  Iter itr;
  // 手牌のパターン数
  long long dens;
  // 七対子を構成する牌の種類数
  int kind_sp;
  // 七対子を構成する対子の種類数
  int pair_sp;
  // 国士無双を構成する牌の種類数
  int kind_to;
  // 国士無双を構成する対子の種類数
  int pair_to;
};

struct Hash{
  std::size_t operator()(const Key& key) const
  {
    using boost::hash_combine;
    using boost::hash_range;

    std::size_t seed = 0;

    hash_combine(seed, hash_range(key.itr->begin(), key.itr->end()));
    hash_combine(seed, key.kind_sp);
    hash_combine(seed, key.pair_sp);
    hash_combine(seed, key.kind_to);
    hash_combine(seed, key.kind_sp);

    return seed;
  }
};

bool operator==(const Key& lhs, const Key& rhs)
{
  return *lhs.itr == *rhs.itr
    && lhs.dens == rhs.dens
    && lhs.kind_sp == rhs.kind_sp
    && lhs.pair_sp == rhs.pair_sp
    && lhs.kind_to == rhs.kind_to
    && lhs.pair_to == rhs.pair_to;
}

void add1(Vec& lhs, const Vec& rhs, const int m)
{
  for(int j=m+5; j>=5; --j){
    int sht = std::min(lhs[j]+rhs[0], lhs[0]+rhs[j]);

    for(int k=5; k<j; ++k){
      sht = std::min({sht, lhs[k]+rhs[j-k], lhs[j-k]+rhs[k]});
    }
    lhs[j] = sht;
  }
  for(int j=m; j>=0; --j){
    int sht = lhs[j]+rhs[0];

    for(int k=0; k<j; ++k){
      sht = std::min(sht, lhs[k]+rhs[j-k]);
    }
    lhs[j] = sht;
  }
}

void add2(Vec& lhs, const Vec& rhs, const int m)
{
  int j = m+5;
  int sht = std::min(lhs[j]+rhs[0], lhs[0]+rhs[j]);

  for(int k=5; k<j; ++k){
    sht = std::min({sht, lhs[k]+rhs[j-k], lhs[j-k]+rhs[k]});
  }
  lhs[j] = sht;
}

void read_file(Iter first, const Iter last, const char* file)
{
  std::ifstream fin(file);

  int tmp;

  for(; first!=last; ++first){
    for(int j=0; j<10; ++j){
      fin >> tmp;
      (*first)[j] = tmp;
    }
  }
}

// 手牌生成(全幅)
template <int N, bool IsHonor>
void generate(int n, int m, std::vector<Vec>& table, Keys& keys)
{
  // 組み合わせ(4Cr)
  static const Vec c = {1, 4, 6, 4, 1};
  static Vec quin(N+1);
  static Vec dens(N+1, 1);
  static Vec kind_sp(N+1);
  static Vec pair_sp(N+1);
  static Vec kind_to(N+1);
  static Vec pair_to(N+1);

  if(n == N){
    const Key key{
      std::next(table.begin(), quin[N]),
      dens[N],
      kind_sp[N],
      pair_sp[N],
      kind_to[N],
      pair_to[N]
    };

    if(auto itr=keys.find(key); itr == keys.end()){
      keys.emplace(key, 1);
    }
    else{
      ++itr->second;
    }
  }
  else{
    for(int i=std::max(0, m-4*(N-1-n)); i<=std::min(4, m); ++i){
      quin[n+1] = quin[n] * 5 + i;
      dens[n+1] = dens[n] * c[i];
      kind_sp[n+1] = kind_sp[n] + (i > 0 ? 1 : 0);
      pair_sp[n+1] = pair_sp[n] + (i >= 2 ? 1 : 0);
      kind_to[n+1] = kind_to[n] + ((IsHonor || n == 0 || n == N-1) && i > 0 ? 1 : 0);
      pair_to[n+1] = pair_to[n] + ((IsHonor || n == 0 || n == N-1) && i >= 2 ? 1 : 0);

      generate<N, IsHonor>(n+1, m-i, table, keys);
    }
  }
}

inline int coef(const int i, const int j, const int k)
{
  if(i == j && j == k){
    return 1;
  }
  else if(i == j || j == k || k == i){
    return 3;
  }
  else{
    return 6;
  }
}

int main()
{
  constexpr int M = 14;

  const auto start = std::chrono::system_clock::now();

  std::vector<Vec> mp1(1953125, Vec(10));
  std::vector<Vec> mp2(78125, Vec(10));

  read_file(mp1.begin(), mp1.end(), "index_s.txt");
  read_file(mp2.begin(), mp2.end(), "index_h.txt");

  std::vector<Keys> keys1(M+1);
  std::vector<Keys> keys2(M+1);

  for(int m=0; m<=M; ++m){
    generate<9, false>(0, m, mp1, keys1[m]);
    generate<7, true>(0, m, mp2, keys2[m]);
  }

  std::valarray<long long> cnt(8);

  for(int i=0; i<=M; ++i){
    for(int j=i; j<=M-i; ++j){
      for(int k=j; k<=M-i-j; ++k){
        std::valarray<long long> tmp(8);

        for(const auto& [key0, value0] : keys1[i]){
          for(const auto& [key1, value1] : keys1[j]){
            for(const auto& [key2, value2] : keys1[k]){
              for(const auto& [key3, value3] : keys2[M-i-j-k]){
                Vec lhs = *key0.itr;
                add1(lhs, *key1.itr, M/3);
                add1(lhs, *key2.itr, M/3);
                add2(lhs, *key3.itr, M/3);

                const auto dens = key0.dens*value0 * key1.dens*value1 * key2.dens*value2 * key3.dens*value3;
                const auto kind_sp = key0.kind_sp + key1.kind_sp + key2.kind_sp + key3.kind_sp;
                const auto pair_sp = key0.pair_sp + key1.pair_sp + key2.pair_sp + key3.pair_sp;
                const auto kind_to = key0.kind_to + key1.kind_to + key2.kind_to + key3.kind_to;
                const auto pair_to = key0.pair_to + key1.pair_to + key2.pair_to + key3.pair_to;

                const int num_lh = lhs[5 + M/3];
                const int num_sp = 7 - pair_sp + (kind_sp < 7 ? 7 - kind_sp : 0);
                const int num_to = 14 - kind_to - (pair_to > 0 ? 1 : 0);
                const int num_all = std::min({num_lh, num_sp, num_to});

                tmp[num_all] += dens;
              }
            }
          }
        }

        cnt += tmp * coef(i, j, k);
      }
    }
  }

  const auto end = std::chrono::system_clock::now();
  const auto total = cnt.sum();
  double ev = 0.0;

  std::cout << "Shanten\tCount\tRate" << std::endl;

  for(int i=0; i<8; ++i){
    ev += (i - 1) * cnt[i];
    std::cout << i - 1 << "\t" << cnt[i] << "\t" << static_cast<double>(cnt[i]) / total << std::endl;
  }

  std::cout << "Number of Tiles: " << M << std::endl;
  std::cout << "Total: " << total << std::endl;
  std::cout << "Time (msec.): " << std::chrono::duration_cast<std::chrono::milliseconds>(end - start).count() << std::endl;
  std::cout << "Expected Value: " << ev / total << std::endl;

  return 0;
}
```

出力:

```
Shanten Count   Rate
-1      12859078207674  3.02545e-06
0       2966241795738948        0.000697889
1       99154452630748356       0.0233288
2       828714358375292670      0.194978
3       1867404976243926528     0.439358
4       1211948980271480832     0.285144
5       233501763289743360      0.0549376
6       6601397483077632        0.00155316
Number of Tiles: 14
Total: 4250305029168216000
Time (msec.): 1036
Expected Value: 3.15594
```
