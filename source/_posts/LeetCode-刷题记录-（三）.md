---
title: LeetCode-刷题记录（三）
date: 2017-09-20 10:53:18
tags:
        - LeetCode
---

# Interleaving String

> https://leetcode.com/problems/interleaving-string/description/

## 题意

给定三个字符串s1、s2、s3，判断s3是否是由s1与s2交错构成。

比如`"abc"`就是由`"ac"`与`"b"`交错构成的，而`"cab"`不是。

<!-- more -->

## 思路

用类似动态规划的思路，`dp[i][j]`表示是否存在一个交错方式使得`s1`的前`i`位与`s2`的前`j`位交错能够生成`s3`的前`i+j`位。

于是有这样的状态关系：`dp[i][j] = (dp[i-1][j] && s1[i] == s3[i+j]) || (dp[i][j-1] && d2[j] == s3[i+j])`。

实现一下递推就好了，一般可以用记忆化或者类似BFS的递推形式，下面我用的是后一种方法。

## 代码

```c++
class Solution {
public:
    bool isInterleave(string s1, string s2, string s3) {
        if (s1.size() + s2.size() != s3.size()) return false;
        vector<vector<bool>> dp;
        dp.resize(s1.size() + 1);
        for (auto it = dp.begin(); it != dp.end(); it++) it->resize(s2.size() + 1, false);
        dp[0][0] = true;

        list<pair<int, int>> li = { {0, 0} };
        while (!li.empty()) {
            auto p = li.front();
            li.pop_front();
            dp[p.first][p.second] = true;
            if (p.first + p.second + 1 > s3.size()) continue;
            char next = s3[p.first + p.second];
            if (p.first < s1.size() && s1[p.first] == next) {
                if (!dp[p.first + 1][p.second]) {
                    dp[p.first + 1][p.second] = true;
                    li.push_back({ p.first + 1, p.second });
                }
            }
            if (p.second < s2.size() && s2[p.second] == next) {
                if (!dp[p.first][p.second + 1]) {
                    dp[p.first][p.second + 1] = true;
                    li.push_back({ p.first, p.second + 1 });
                }
            }
        }

        return dp[s1.size()][s2.size()];
    }
};
```
