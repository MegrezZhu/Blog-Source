---
title: LeetCode 刷题记录 （十三）
date: 2017-12-13 14:25:16
tags:
  - LeetCode
---

# Longest Increasing Path in a Matrix

> https://leetcode.com/problems/longest-increasing-path-in-a-matrix/description/

## 题意

给定一个$n\times m$的整数矩阵$A$，找出里面最长的上升路径。（只能走上下左右四个方向）

<!-- More -->

## 思路

大体上还是与一维数组中的最长上升子序列相似，又因为是上升路径而不是不下降路径因此这个路径是不可能有环的。于是以$F_{i,j}$表示以$(i,j)$为终点的最长上升路径的长度，$D_4(i,j)$表示点$(i,j)$的*4-邻域*，于是有如下转移公式：
$$
F_{i,j}=max(\{0\}\cup\{F_{i',j'}|(i',j')\in D(i,j)\land A_{i,j}>A_{i',j'}\})+1
$$
本题与一维数组的最长上升子序列计算的一个重要区别在于：一个路径并不是简单地“从左到右”的。

注意到上面的状态转移公式中，对于矩阵中的一个坐标$(i,j)$，为了计算$F_{i,j}$，需要所有满足$(i',j')\in D(i,j)\land A_{i,j}>A_{i',j'}$的点（与$(i,j)$相邻且更小的点）的$F$计算出来。于是就涉及到一个计算顺序的问题：我们需要按照$A_{i,j}$升序顺序计算$F_{i,j}$，而不是简单的（如一维数组问题中的）坐标顺序。

**解决方案**

1. 首先当然可以$O((mn)log_2(mn))$做一次排序后按照排序结果的顺序进行计算，但计算每个$F_{i,j}$只需要$O(1)$的复杂度，因此排序的速度已经成为整体复杂度的瓶颈。
2. 于是可以采用类似记忆化搜索的递归过程，记录每个$F_{i,j}$是否已经计算，而当计算$F_{i,j}$依赖的$F_{i',j'}$的结果并未被计算时进行递归计算、否则直接使用保存的值。这样整体复杂度降为了$O(mn)$。

## 代码

```c++
class Solution {
    static int dx[4];
    static int dy[4];

  int dp(const vector<vector<int>>& matrix, vector<vector<int>>& result, int x, int y) {
    if (result[x][y] > 0) return result[x][y];

    int n = matrix.size(), m = matrix.front().size();
    result[x][y] = 1;
    for (int dir = 0; dir < 4; dir++) {
      int tx = x + dx[dir], ty = y + dy[dir];
      if (tx < 0 || tx >= n || ty < 0 || ty >= m || matrix[tx][ty] <= matrix[x][y]) continue;
      result[x][y] = max(result[x][y], dp(matrix, result, tx, ty) + 1);
    }

    return result[x][y];
  }
public:
  int longestIncreasingPath(vector<vector<int>>& matrix) {
    if (matrix.empty()) return 0;

    int n = matrix.size(), m = matrix.front().size();
    vector<vector<int>> results(n, vector<int>(m, -1));
    int ans = 0;
    for (int i = 0; i < n; i++) {
      for (int j = 0; j < m; j++) ans = max(ans, dp(matrix, results, i, j));
    }

    return ans;
  }
};

int Solution::dx[4] = { 0, 1, 0, -1 };
int Solution::dy[4] = { 1, 0, -1, 0 };
```



# Smallest Good Base

> https://leetcode.com/problems/smallest-good-base/description/

## 题意

对于一个整数$n$，它的一个*Good Base*（好基？）被定义为一个这样的一个整数$k$，满足：

* $k\ge 2$
* 在$k$进制下，$n$的每一位都是$1$

于是问题是，给定一个字符串表示$n$，$n\in[3,10^{18}]$，求它的最小的*Good Base*。

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
