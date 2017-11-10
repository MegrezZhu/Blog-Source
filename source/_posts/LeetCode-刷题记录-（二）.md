---
title: LeetCode 刷题记录 （二）
date: 2017-09-12 15:40:04
tags:
    - LeetCode
---

# Regular Expression Matching

> https://leetcode.com/problems/regular-expression-matching/description/

## 题意

给定一个正则表达式，和一个字符串，判断给定的正则表达式是否与给定的字符串匹配。

这里的正则表达式是简化版的，只有`.` 与`*`两种元字符，而且只要求判断是否整体匹配（即整个字符串都是匹配的），所以比实现一个完整的字符串简单多了。

<!-- more -->

## 非常暴力的做法

直接上带正则表达式支持的语言就好（逃

比如**JavaScript**：

```javascript
/**
 * @param {string} s
 * @param {string} p
 * @return {boolean}
 */
const isMatch = (s, p) => !!s.match(new RegExp(`^${p}$`));
```

完事。

## 没那么暴力的做法

### 解题思路

记待匹配串为s，正则表达式串为p，n为s的长度，m为p的长度。

记n、m分别为待匹配串与正则表达式串的长度， 并级。

以类似动态规划的思路，以`f[i][j]`表示字符串匹配到第i位、正则串匹配到第j位的可行性（那么最终的匹配结果就是`f[n][m]`）。因此一共有mn个状态。在最坏情况下，需要把所有状态的结果都计算出来，因此时间复杂度为`O(mn)`。

状态的转移相对比较简单：

* 对于最简单的字符匹配，有`f[i + 1][j + 1] = f[i][j]`（`s[i + 1] == p[j + 1]`时）
* 对于`.*`与`c*`的匹配，有`f[i + k][j + 2] = f[i][j]`（其中`k = 0, 1, ...`，需满足`s[i + 1] == s[i + 2] == ... == s[i + k] = c` 或 `c == '.'` 。其中c为被`*` 修饰的待匹配字符）

具体实现见后面的代码。

### 坑点

边界条件比较复杂。

### 代码

```c++
class Solution {
public:
    bool isMatch(string ss, string pp) {
        s = ' ' + ss;
        p = ' ' + pp;
        res.resize(s.size());
        calculated.resize(s.size());
        for (int i = 0; i < s.size(); i++) {
            // 初始化
            res[i].clear();
            res[i].resize(p.size(), false);
            calculated[i].clear();
            calculated[i].resize(p.size(), false);
        }

        match(0, 0);
        return res[s.size() - 1][p.size() - 1];
    }

    void match(int i, int j) {
        if (i >= s.size() || j >= p.size()) return;
        if (calculated[i][j]) return;
        res[i][j] = calculated[i][j] = true;

        if (j < p.size() - 1) { // regexp remained
            bool starMatch = false;
            if (j < p.size() - 2 && p[j + 2] == '*') starMatch = true;
            if (i < s.size() - 1) {
                char toMatch = p[j + 1];
                bool anyMatch = toMatch == '.';
                if (starMatch) {
                    int k = i;
                    while (k < s.size()) {
                        match(k, j + 2);
                        if (k < s.size() - 1 && (anyMatch || s[k + 1] == toMatch)) k++;
                        else break;
                    }
                }
                else {
                    if (anyMatch || s[i + 1] == toMatch) match(i + 1, j + 1);
                }
            }
            else {
                if (starMatch) match(i, j + 2);
            }
        }
    }

private:
    string s, p;
    vector<vector<bool>> res, calculated;
};
```
