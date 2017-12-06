---
title: LeetCode 刷题记录 （十二）
date: 2017-12-06 15:44:08
tags:
	- LeetCode
---

# Russian Doll Envelopes

> https://leetcode.com/problems/russian-doll-envelopes/description/

## 题意

有$n$个信封，每个信封都有自己的宽高，并且当一个信封$E_a$的宽高均严格大于另一个信封$E_b$的宽高时，可以将$E_b$套入$E_b$中。

现在给定若干个信封的宽高，问最多可以有几层的嵌套信封（原文是“俄罗斯套娃”）。

<!-- More -->

## 思路

首先这道题的模型与最长上升子序列很像：

* 都有一个小于关系
* 都存在一个这样的性质：当$A$“大于”$B$时，$A$也“大于”$B$所嵌套的“信封”

**于是就有一个Naive的做法：**

1. 把信封按宽度递增、再按高度递增进行排序
2. $F[E_i]$表示信封$i$为最外层信封时的最大嵌套层数，则有$F[E_i]=max\{F[E_j]+1|E_i>E_j\}$

注意：就算排序之后，出现在后面的信封也不一定能嵌套前面的信封。

大致代码如下：

```c++
for (int i = 1; i < n; i++) {
  for (int j = 0; j < i; j++) {
    if (E[i] < E[j]) { // need check
      dp[i] = max(dp[i], dp[j] + 1);
    }
  }
}
```

时间复杂度为$O(N^2)$，主要是没办法利用上经典的最长上升子序列的优化。

**厉害一点的做法**

考虑改变一种排序的方式：按宽度递增、在宽度相等时按高度递减排序，于是此时出现在$(w,h)$前面的任意元素$(w',h')$只有这两种情况之一：

* $w'=w\land h'=h$，高度相等、因而不能嵌套
* $w'<w'$，是否可以嵌套取决于高度

于是可以按高度做一次最长上升子序列，加上如下优化：

* 维护$M_i$，表示所有满足长度为$i$的嵌套信封方案当中，最外层信封所需高度的最小值。
* 对于一个高度为$h$的信封，二分查找最大的$i$满足$M_i\lt h$，然后更新$M_{i+1}$

## 代码

```c++
class Solution {
  bool less(const pair<int, int>& a, const pair<int, int>& b) {
    return a.first < b.first && a.second < b.second;
  }
public:
  int maxEnvelopes(vector<pair<int, int>>& envelopes) {
    int n = envelopes.size();
    if (!n) return 0;
    sort(envelopes.begin(), envelopes.end(), [](const pair<int, int>& a, const pair<int, int>& b) -> bool {
      if (a.first > b.first) return false;
      if (a.first < b.first) return true;
      if (a.second > b.second) return true;
      if (a.second < b.second) return false;
      return false;
    });
    int res = 1;
    vector<int> mx(n + 1, 2e9);
    mx[0] = 0;
    for (int i = 0; i < n; i++) {
      int l = 0, r = n;
      while (l < r) {
        int m = (l + r + 1) / 2;
        int mid = mx[m];
        if (mid < envelopes[i].second) l = m;
        else r = m - 1;
      }

      mx[l + 1] = min(envelopes[i].second, mx[l + 1]);

      res = max(res, l + 1);
    }
    return res;
  }
};
```

