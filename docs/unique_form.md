# 固有形

## 定義

**固定面子**
: $3N+1$(または$3N+2$)聴牌形$h \in H$に対して, その面子を除いても聴牌形を維持し待ちを不変に保つならばこれを固定面子という. 

固定面子を用いて以下の3種類の固有形を定義する.

**$3N+1$聴牌固有形**
: $h \in H$が$3N+1$聴牌形で固定面子を持たないならば, $h \in H$は$3N+1$聴牌固有形である. 

**$3N+2$聴牌固有形**
: $h^n \in H^n$が$3N+2$聴牌形であるが$3N+2$和了形ではなく固定面子を持たないならば, $h^n \in H^n$は$3N+2$聴牌固有形である. 

**$3N+2$和了聴牌固有形**
: $h^n \in H^n$が$3N+2$聴牌形であると同時に$3N+2$和了形であり固定面子を持たないならば, $h^n \in H^n$は$3N+2$和了聴牌固有形である. 

## 固有形の分類

任意の$3N+1$聴牌固有形は以下のいずれかのパターンに分類される.

- $3N+2$聴牌固有形1つと雀頭の組み合わせ(**雀頭固定型**)
- $3N+2$和了聴牌固有形2つの組み合わせ(**シャボ型**)
- 上記以外(**一体型**)

## ソースコード

### 3N+1聴牌固有形判定

```cpp
bool isuf1(int* h)
{
  auto wait = isrh1(h);

  for(int i=0; i<9; ++i){
    if(h[i] >= 3){
      h[i] -= 3;
      auto tmp = isrh1(h);
      h[i] += 3;

      if(((~wait|tmp)&((1<<9)-1)) == (1<<9)-1){
        return false;
      }
    }
  }
  for(int i=0; i<7; ++i){
    if(h[i] >= 1 && h[i+1] >= 1 && h[i+2] >= 1){
      --h[i]; --h[i+1]; --h[i+2];
      auto tmp = isrh1(h);
      ++h[i]; ++h[i+1]; ++h[i+2];

      if(((~wait|tmp)&((1<<9)-1)) == (1<<9)-1){
        return false;
      }
    }
  }
  return true;
}
```

### 3N+2聴牌固有形判定

```cpp
bool isuf2(int* h)
{
  auto wait = isrh2(h);

  for(int i=0; i<9; ++i){
    if(h[i] >= 3){
      h[i] -= 3;
      auto tmp = isrh2(h);
      h[i] += 3;

      if(((~wait|tmp)&((1<<9)-1)) == (1<<9)-1){
        return false;
      }
    }
  }
  for(int i=0; i<7; ++i){
    if(h[i] >= 1 && h[i+1] >= 1 && h[i+2] >= 1){
      --h[i]; --h[i+1]; --h[i+2];
      auto tmp = isrh2(h);
      ++h[i]; ++h[i+1]; ++h[i+2];

      if(((~wait|tmp)&((1<<9)-1)) == (1<<9)-1){
        return false;
      }
    }
  }
  return true;
}
```

### 3N+2和了聴牌固有形判定

```cpp
bool iswr2(int* h)
{
  auto wait = isrh2(h);

  for(int i=0; i<9; ++i){
    if(h[i] >= 3){
      h[i]-=3;
      auto tmp = isrh2(h);

      if(((~wait|tmp)&((1<<9)-1)) == (1<<9)-1 && iswh2(h)){
        h[i] += 3;
        return false;
      }
      else h[i] += 3;
    }
  }
  for(int i=0; i<7; i++){
    if(h[i] >= 1 && h[i+1] >= 1 && h[i+2] >= 1){
      --h[i]; --h[i+1]; --h[i+2];
      auto tmp = isrh2(h);

      if(((~wait|tmp)&((1<<9)-1)) == (1<<9)-1 && iswh2(h)){
        ++h[i]; ++h[i+1]; ++h[i+2];
        return false;
      }
      else{
        ++h[i]; ++h[i+1]; ++h[i+2];
      }
    }
  }
  return true;
}
```
