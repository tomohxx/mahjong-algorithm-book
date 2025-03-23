# 手牌(牌山)生成

## 全幅

$M$枚の牌からなる手牌, すなわち$\sum_{i=0}^8 h^n_i = M$となるすべての$h^n$を生成する. 今$h^n_i$を考えていて, ($i$番目の牌を含め)$m$枚牌を使わなければならないとする. まず$h^n_i$の最大値は${\rm min}(4, m)$である. 次に最小値を考える. $i$番目の牌の枚数を決めた後は$4(8-i)$枚が未使用になっている. $m$がこの値より大きいときはその差を最小値とする必要がある. よって$h^n_i$の最小値は${\rm max}(0, m-4(8-i))$である. $h^n_{i+1}$については$m$を$m-h^n_i$に置き換えて同様に考えればよい. この処理を再帰関数で実装する.

```cpp
#include <iostream>
#include <functional>
#include <vector>

void generate(int n, int m, std::function<void(const std::vector<int>&)> func)
{
  static std::vector<int> h(9);

  if (n == 9) {
    func(h);
  }
  else {
    for (int i = std::max(0, m - 4 * (8 - n)); i <= std::min(4, m); ++i) {
      h[n] = i;
      generate(n + 1, m - i, func);
    }
  }
}

int main()
{
  int M;

  std::cin >> M;

  generate(0, M, [](const std::vector<int>& h) {
    for (const auto& i : h) {
      std::cout << i << ' ';
    }
    std::cout << '\n';
  });

  return 0;
}
```

## 乱択

$M$枚の牌からなる手牌をランダムに$N$個生成する. これは[Fisher-Yates 法](https://ja.wikipedia.org/wiki/%E3%83%95%E3%82%A3%E3%83%83%E3%82%B7%E3%83%A3%E3%83%BC%E2%80%93%E3%82%A4%E3%82%A7%E3%83%BC%E3%83%84%E3%81%AE%E3%82%B7%E3%83%A3%E3%83%83%E3%83%95%E3%83%AB)を用いて実現できる. ここでは 136 枚の牌を区別して手牌を構成する牌の番号を出力する.

```cpp
#include <iostream>
#include <random>
#include <vector>

int main()
{
  int M, N;
  std::vector<int> tiles(136);
  std::mt19937 rand(std::random_device{}());

  std::cin >> M >> N;

  std::iota(tiles.begin(), tiles.end(), 0);

  for (int i = 0; i < N; ++i) {
    for (int j = 0; j < M; ++j) {
      int n = rand() % (136 - j);
      std::cout << tiles[n] << " ";
      std::swap(tiles[n], tiles[135 - j]);
    }
    std::cout << "\n";
  }

  return 0;
}
```

!!! note

    `std::sample`の利用も検討する

## 重複組み合わせ

同じ種類の牌を区別する場合の 14 枚の手牌のパターン数は$_{136}C_{14}$である. 一方, 同じ種類の牌を区別しない場合のパターン数は, すべての手牌パターンを生成することでも求められるが, 重複組み合わせを考えれば高速に求められる. アルゴリズムについては『プログラミングコンテストチャレンジブック 第 2 版』を参照することとし, ここではソースコードのみ示す.

```cpp
#include <array>
#include <iostream>

int main()
{
  // N 種類の牌があり, それぞれの種類の牌は A 枚ずつある.
  // 異なる種類の牌同士は区別できるが, 同じ種類の牌同士は区別できない.
  // これらの牌から M 枚選ぶ組み合わせの総数を求める.

  constexpr int N = 34;
  constexpr int M = 14;
  constexpr int A = 4;

  std::array<std::array<unsigned long long, M + 1>, N + 1> dp{};

  for (int i = 0; i <= N; ++i) {
    dp[i][0] = 1;
  }

  for (int i = 0; i < N; ++i) {
    for (int j = 1; j <= M; ++j) {
      if (j - 1 - A >= 0) {
        dp[i + 1][j] = dp[i + 1][j - 1] + dp[i][j] - dp[i][j - 1 - A];
      }
      else {
        dp[i + 1][j] = dp[i + 1][j - 1] + dp[i][j];
      }
    }
  }

  std::cout << dp[N][M] << std::endl;

  return 0;
}
```

出力:

```
326520504500
```
