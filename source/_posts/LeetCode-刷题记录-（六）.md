---
title: LeetCode 刷题记录 （六）
date: 2017-10-13 15:19:51
tags:
    - LeetCode
---

# Candy

> https://leetcode.com/problems/candy/description/

## 题意

$N$个小朋友站成一行，每个小朋友有一个评级$R_i$。现在要给每个小朋友派发糖果，第$i$个小朋友得到的糖果的数量$C_i$须满足如下条件：

* $C_i\ge1$
* 对于$|i-j|=1$，如果$R_i>R_j$，则$C_i>C_j$

计算最少需要的糖果总数。

<!-- more -->

## 思路

根据要求递归计算即可，加上记忆化防止重复计算。时间复杂度为$\mathcal{O}(N)$。

## 代码

```c++
class Solution {
    vector<int> res;
    vector<int> ratings;
    int calculate(int pos) {
        if (pos < 0 || pos >= res.size()) return 0;
        if (res[pos] != -1) return res[pos];
        int left = get(pos - 1) < get(pos) ? calculate(pos - 1) : 0;
        int right = get(pos + 1) < get(pos) ? calculate(pos + 1) : 0;
        return res[pos] = max(left, right) + 1;
    }
    int get(int pos) {
        return (pos < 0 || pos >= ratings.size()) ? 1e9 : ratings[pos];
    }
public:
    int candy(vector<int>& ratings) {
        this->ratings = ratings;
        res.clear();
        res.resize(ratings.size(), -1);
        int ans = 0;
        for (int i = 0; i < res.size(); i++) ans += calculate(i);
        return ans;
    }
};
```

# Maximum Subarray

> https://leetcode.com/problems/maximum-subarray/description/

## 题意

给定一个数组$A$，求它的最大子段和。

## 思路

经典的DP题。

$F_i$表示以$A_i$结尾的所有子段中能够得到的最大字段和，于是有：
$$
F_i=\max\{F_{i-1},0\}+A_i
$$
初始有$F_0=A_0$。

时间复杂度为$\mathcal{O}(N)$，$N$为数组中元素个数。

## 代码

```c++
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        vector<int> f;
        f.resize(nums.size());
        f[0] = nums[0];
        int ans = f[0];
        for (int i = 1; i < nums.size(); i++) {
            f[i] = max(f[i - 1], 0) + nums[i];
            ans = max(ans, f[i]);
        }
        return ans;
    }
};
```
