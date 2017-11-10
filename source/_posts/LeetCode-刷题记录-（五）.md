---
title: LeetCode 刷题记录 （五）
date: 2017-10-11 14:38:47
tags:
        - LeetCode
---

# Dungeon Game

> https://leetcode.com/problems/dungeon-game/description/

## 题意
一个$m\times n$的矩阵当中，一个骑士从左上角走到右下角，只能向右或向下移动。骑士有初始血量$H$，矩阵的每一格中有一个数值$A_{i,j}$表示移动到这一格时对血量的增减，计算能保证不在任意时刻血量降至0以下的最小初始血量$H_{min}$。

<!-- more -->

## 思路

因为只能向右或下移动，因此避免了走回路的情况。记$F_{i,j}$为走到以第$i$行第$j$列（从0开始）为起点，走到终点要求的最小初始血量。显然地，$F_{i,j}$只与$F_{i,j+1}$、$F_{i+1,j}$以及$A_{i,j}$相关。于是有下面的关系：
$$
F_{i,j}=max\{0,-A_{i,j}+min\{F_{i,j+1},F_{i+1,j}\}\}
$$

特别地，对于终点有：
$$
F_{m-1,n-1}=max\{0, -A_{m-1,n-1}\}
$$
上面的递推保证了在任意时刻血量均不会降为负数，因为题目里需要正数血量，因此最后的答案就是$F_{0,0}+1$。

## 代码

```c++
class Solution {
public:
    int calculateMinimumHP(vector<vector<int>>& dungeon) {
        int n = dungeon.size(), m = dungeon.front().size();
        vector<vector<int>> res;
        res.resize(n + 1);
        for (auto it = res.begin(); it != res.end(); it++) it->resize(m + 1, 1e9);
        res[n - 1][m - 1] = max(-dungeon[n - 1][m - 1], 0);
        for (int i = n - 1; i >= 0; i--) {
            for (int j = m - 1; j >= 0; j--) {
                if (i == n - 1 && j == m - 1) continue;
                res[i][j] = -dungeon[i][j] + min(res[i][j + 1], res[i + 1][j]);
                res[i][j] = max(res[i][j], 0);
            }
        }
        return res[0][0] + 1;
    }
};
```

# Shortest Palindrome

> https://leetcode.com/problems/shortest-palindrome/description/

## 题意

给定一个字符串$S$，在它前面增加若干字符得到$S'$使得它成为一个回文字符串，求满足要求的最短的$S'$。

## 思路

如果能将$S$分解为$S_1S_2$，其中$S_1$为回文串，那么只需要在$S_1S_2$前加上$S_2$的翻转$S'_2$，得到的$S'_2S_1S_2$就是一个回文字符串。

问题变为寻找一个最长的回文子串$S_1$。

这是一个经典的问题，可以使用[Manacher](https://segmentfault.com/a/1190000003914228)算法解决。

## 代码

```c++
class Solution {
    static string preprocess(const string& s) {
        string res;
        res.resize(s.size() * 2 + 1);
        for (int i = 0; i < res.size(); i++) {
            if (i % 2) res[i] = s[i / 2];
            else res[i] = '#';
        }
        return res;
    }

    static int mirror(int pos, int middle) {
        return 2 * middle - pos;
    }

    static int longestPrefixingPalidrome(const string& s) {
        string ss = preprocess(s);
        vector<int> rl = { 0 };
        int maxRight = 0, pos = 0, ans = 0;

        for (int i = 1; i < ss.length(); i++) {
            int j;
            if (i > maxRight) j = i;
            else {
                // i <= maxRight
                int knownLen = rl[mirror(i, pos)];
                j = min(i + knownLen, maxRight);
            }
            while (j < ss.length() && mirror(j, i) >= 0 && ss[j] == ss[mirror(j, i)]) j++;
            j--;
            rl.push_back(j - i);
            if (j > maxRight) {
                maxRight = j;
                pos = i;
            }
            if (mirror(j, i) == 0) ans = max(ans, rl[i]);
        }

        return ans;
    }
public:
    string shortestPalindrome(string s) {
        int len = longestPrefixingPalidrome(s);
        auto left = s.substr(len);
        reverse(left.begin(), left.end());
        return left + s;
    }
};
```
