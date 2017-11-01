---
title: LeetCode 刷题记录 （九）
date: 2017-11-01 15:49:46
tags:
    - LeetCode
---

# K Inverse Pairs Array

> https://leetcode.com/problems/k-inverse-pairs-array/description/

## 题意

给定$n$和$k$，计算由$1$到$n$的$n$个正整数组成的所有序列中，恰有$k$对逆序对的序列个数。（因为答案较大，因此只需计算答案模$10^9+7$的余数。

## 思路

考虑一个这样的序列，它由$1$~$n_1$这$n_1$个正整数组成。此时把一个$n_1+1$插入序列中，显然得到的长为$n_1+1$的新序列中的逆序对个数仅与原序列中的逆序对个数、还有插入的$n_1+1$的位置有关（因为它比原序列中的任何数都大，因此在插入的位置后面的原序列当中的数字都会与$n_1+1$形成一个新的逆序对）。

于是得到状态转移：
$$
F_{i,j}=(\sum_{k\ge j-i+1\land k\ge0}^{}{F_{i-1,k}}) \% (10^9 + 7)
$$
$F_{i,j}$表示长度为$i$的序列中，满足有$j$个逆序对的解的数量（模$10^9+7$）。

然后加上和背包问题中类似的减少需要的内存空间的方法，可以在$O(n)$的时空复杂度下解决问题。

## 代码

```c++
class Solution {
public:
    int kInversePairs(int n, int k) {
        constexpr int MOD = 1e9 + 7;

        vector<int> f(k + 1);
        f[0] = 1;

        for (int i = 2; i <= n; i++) {
            int p = k + 1, sum = 0;
            for (int j = k; j > 0; j--) {
                while (p - 1 >= 0 && p - 1 >= j - i + 1) sum = (sum + f[--p]) % MOD;
                int tmp = f[j];
                f[j] = sum;
                sum = (sum + MOD - tmp) % MOD;
            }
        }

        return f[k];
    }
};

```
