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

$M$枚の牌からなる手牌をランダムに$N$個生成する. これは[Fisher-Yates法](https://ja.wikipedia.org/wiki/%E3%83%95%E3%82%A3%E3%83%83%E3%82%B7%E3%83%A3%E3%83%BC%E2%80%93%E3%82%A4%E3%82%A7%E3%83%BC%E3%83%84%E3%81%AE%E3%82%B7%E3%83%A3%E3%83%83%E3%83%95%E3%83%AB)を用いて実現できる. ここでは136枚の牌を区別して手牌を構成する牌の番号を出力する.

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
      std::swap(tiles[n], tiles[135 - j]);
      std::cout << n << " ";
    }
    std::cout << "\n";
  }

  return 0;
}
```

!!! Note
    `std::sample`の利用も検討する
