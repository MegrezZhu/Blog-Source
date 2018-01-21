---
title: LeetCode 刷题记录 （十六）
date: 2017-12-29 20:11:06
tags:
  - LeetCode
---

# Distinct Subsequences

> https://leetcode.com/problems/distinct-subsequences/description/

## 题意

对于给定的两个字符串$S$和$T$，求$S$有多少个子序列等于$T$。

<!-- More -->

## 思路

动态规划。$F_{i,j}$表示$S_{0:i}$（表示$S$下标从$0$到$i$的前缀）中等于$T_{0..j}$的子序列个数，于是有转移方程：
$$
F_{i,j}=F_{i-1,j}+F_{i-1,j-1}\times(S_{i}=T_{j})
$$
再应用类似01背包问题中的优化，可以将状态从二维降为一维。具体实现见下面代码。

最终时间复杂度为$O(|S||T|)$，空间复杂度为$O(|T|)$

## 代码

```c++
class Solution {
public:
  int numDistinct(string s, string t) {
    vector<int> f;
    f.resize(t.length() + 1, 0);
    f[0] = 1; // dp[0][0] = 1
    for (int i = 1; i <= s.length(); i++) {
      for (int j = t.length(); j > 0; j--) {
        if (i < j) f[j] = 0;
        else f[j] = f[j] + (s[i - 1] == t[j - 1] ? f[j - 1] : 0);
      }
    }
    return f[t.size()];
  }
};
```
