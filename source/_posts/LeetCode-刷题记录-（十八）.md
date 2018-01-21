---
title: LeetCode 刷题记录 （十八）
date: 2018-01-02 10:14:17
tags:
	- LeetCode
---

# Decode Ways II

> https://leetcode.com/problems/decode-ways-ii/description/

## 题意

一个只含有大写英文字母的字符串，根据`'A' -> 1; 'B' -> 2; ...'Z' -> 26`的规则被编码为只含有数字的字符串（如`ABC`被编码为`123`）。

现在给定给一个只含有数字和`*`（通配符，可以是任何字符）的字符串$T$，求有多少个不同的原字符串可以编码成$T$。

<!-- More -->

## 思路

动态规划。$F_{i}$表示$T_{0:i}$可以由多少个不同的原字符串编码而成。于是有下面的状态转移方程：
$$
F_{i}=F_{i-1}G(T_i)+F_{i-2}G(T_{i-1:i})
$$
其中$G(s)$表示数字字符串$s$可以解码为多少种原字符串。

比如：$G('2')=|\{'B'\}|=1$，$G('11')=|\{‘AA','K'\}|=2$，$G('*')=|\{'A', ...,'Z'\}|=26$。

## 代码

```c++
class Solution {
    inline bool isNonZero(char a) {
        return a > '0' && a <= '9';
    }
    inline bool isNum(char a) {
        return a >= '0' && a <= '9';
    }
public:
    int numDecodings(string s) {
        constexpr long long MOD = 1e9 + 7;
        if (!s.length()) return 0;
        vector<long long> f(s.length() + 1);
        f[0] = 1;
        for (int i = 0; i < s.length(); i++) {
            if (s[i] == '*') f[i + 1] = (f[i] * 9) % MOD;
            else if (isNonZero(s[i])) f[i + 1] = f[i];
            else f[i] = 0;

            if (i == 0) continue;
            if (isNonZero(s[i - 1])) {
                if (isNum(s[i])) {
                    if (s[i - 1] * 10 + s[i] - 11 * '0' <= 26) f[i + 1] = (f[i + 1] + f[i - 1]) % MOD;
                } else if (s[i] == '*') {
                    int fac = 0;
                    if (s[i - 1] == '1') fac = 9;
                    else if (s[i - 1] == '2') fac = 6;
                    f[i + 1] = (f[i + 1] + fac * f[i - 1]) % MOD;
                }
            } else if (s[i - 1] == '*') {
                if (s[i] == '*') f[i + 1] = (f[i + 1] + f[i - 1] * 15) % MOD; // 11~19, 21~26
                else if (isNum(s[i])) {
                    f[i + 1] = (f[i + 1] + (s[i] <= '6' ? 2 : 1) * f[i - 1]) % MOD;
                }
            }
        }
        return f[s.length()];
    }
};

```
