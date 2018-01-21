---
title: LeetCode 刷题记录（十五）
date: 2017-12-27 15:32:25
tags:
    - LeetCode
---

# Smallest Good Base

> https://leetcode.com/problems/smallest-good-base/description/

## 题意

对于一个整数$n$，它的一个*Good Base*（好基？）被定义为一个这样的一个整数$k$，满足：

* $k\ge 2$
* 在$k$进制下，$n$的每一位都是$1$

于是问题是，给定一个字符串表示$n$，$n\in[3,10^{18}]$，求它的最小的*Good Base*。

<!-- More -->

## 思路

考虑$n$在$k$进制下的表示，因为全都由$1$组成，因此如果我们要求$n$在$k$进制下的$1$的位数是$len$的话，问题就变成寻找一个最小的$k$满足$(A_{len})_{k}=(n)_{10}$，其中$A_{len}$是一个由$len$个$1$组成的数字串。

很显然在$len$不变的前提下，等式左边是关于$k$单调上升的，于是就可以通过二分来求解是否存在满足条件的k。

同时因为$n\in[3,10^{18}]$，因此显然在$k$进制下，$n$的位数（即$len$）不可能超过$\lceil log_210^{18}\rceil$（代码中直接用63作为上限），因此直接枚举$len$计算在不同$len$$下的答案取最小即可。

> 给定一个字符串表示$n$，$n\in[3,10^{18}]$，求它的最小的*Good Base*

## 代码

```c++
class Solution {
  typedef unsigned long long ull;

  /*
    0 for yes, 1 for greater, -1 for smaller
    @pragma len: number of 1s
  */
  int check(ull base, int len, ull target) {
    ull res = 1;
    while (--len) {
      if (ull(res * base) / base != res) return 1; // overflow
      res = res * base + 1;
    }
    if (res > target) return 1;
    if (res < target) return -1;
    return 0;
  }
public:
  string smallestGoodBase(string n) {
    ull target = stoull(n);
    ull ans = -1; // indicate max

    for (int len = 1; len <= 63; len++) {
      ull l = 2, r = target - 1;
      while (l <= r) {
        ull m = (l + r) / 2;
        int res = check(m, len, target);
        if (res == 0) {
          ans = min(ans, m);
          break;
        }
        if (res == -1) l = m + 1;
        else r = m - 1;
      }
    }

    return to_string(ans);
  }
};
```
