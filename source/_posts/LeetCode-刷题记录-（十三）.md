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
