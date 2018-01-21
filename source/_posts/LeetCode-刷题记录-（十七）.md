---
title: LeetCode 刷题记录 （十七）
date: 2017-12-31 12:01:23
tags:
	- LeetCode
---

# Split Array Largest Sum

> https://leetcode.com/problems/split-array-largest-sum/description/

## 题意

给定一个非负整数数组$A$，将其分割成$m$个非空连续子数组，要求使每个子数组中所有元素的和的最大值最小，并求这个最小的最大值。

<!-- More -->

## 思路

类似于“最大值最小”这样的问题，一般要么是动态规划、要么是二分。

这道题因为满足二分的要求，因此可以二分最大值$M$。对于这个最大值，检查是否可以划分成m个非空数组满足这个最大值，若不满足，则可以确定$M$过大；否则$M$是一个可行的解，于是往更小的方向进行二分。

检查的方法：$O(|A|)$扫描一遍，在满足和不大于$M$的前提下将尽可能多的元素分到一组（这样能保证尽可能少的分组数量），如果这样得出的最少分组数依然大于$m$的话，就可以确定当前选择的$M$过小。

整体复杂度：$O(|A|log_2{(\sum_{i}A_i)})$。

## 代码

```c++
class Solution {
  typedef long long ll;
  bool check(const vector<int> &nums, int m, int limit) {
    for (int i = 0, sum = 0; i < nums.size(); i++) {
      if (sum + nums[i] <= limit) sum += nums[i];
      else {
        m--;
        if (m == 0) return false;
        sum = nums[i];
      }
    }
    return true;
  }
public:
  int splitArray(vector<int>& nums, int m) {
    int n = nums.size();
    int l = 0, r = 0;
    for (int i : nums) {
      l = max(l, i);
      r += i;
    }
    int ans = 2147483647;
    while (l <= r) {
      int mid = (ll(l) + ll(r)) / 2;
      if (check(nums, m, mid)) {
        r = mid - 1;
        ans = min(ans, mid);
      }
      else l = mid + 1;
    }
    return ans;
  }
};

```
