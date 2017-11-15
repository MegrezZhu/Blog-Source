---
title: LeetCode 刷题记录 （十）
date: 2017-11-12 14:46:51
tags:
  - LeetCode
---

# Longest Consecutive Sequence

> https://leetcode.com/problems/longest-consecutive-sequence/description/

## 题意

给定一个未排序的整数数组，找出（排序后）数组里面最长的连续元素序列（如`[2, 3, 4, 5]`）的长度。

## 思路

暴力的做法当然是$O(N\log N)$排序之后$O(N)$扫一遍统计长度，这样总体复杂度是$O(N\log N)$。

机智一些的做法则是$O(N)$的：

<!-- more -->

维护当前已经组成的所有连续序列，每次将一个数组中的元素加入已有序列并更新当前的所有连续序列。扫描一遍原数组需要$O(N)$，那么这个维护过程必须做到$O(1)$才能达到总体$O(N)$的复杂度，具体做法如下：

将一个连续序列表示成一个这样的`<K, V>`对，其中`K`为该连续序列的起点（最小的那个数，最大的也可），`V`为该连续序列的长度。很明显连续序列和`<K, V>`对是一一对应的。

于是就可以用两个`map`结构（为达到$O(1)$水平，C++用`unordered_map<K, V>`， Java用`HashMmap<K, V>`），分别为`left`和`right`，记录下所有以`K`为起点（终点）的连续序列的最大长度`V`。有了这两个`map`，每次添加新元素$X$的时候便可以通过`left`与`right`找到以$X+1$为起点的连续序列的最大长度以及以$X-1$为终点的连续序列的最大长度，显然这两个连续序列加上$X$能组成一个更长的序列。

## 代码

```c++
class Solution {
  void updateMax(unordered_map<int, int> &mp, int pos, int val) {
    auto it = mp.find(pos);
    if (it == mp.end()) mp.insert(make_pair(pos, val));
    else it->second = max(it->second, val);
  }

  void updateMin(unordered_map<int, int> &mp, int pos, int val) {
    auto it = mp.find(pos);
    if (it == mp.end()) mp.insert(make_pair(pos, val));
    else it->second = min(it->second, val);
  }
public:
  int longestConsecutive(vector<int>& nums) {
    unordered_map<int, int> left, right; // left: started a x, leftmost pos
    int ans = 0;
    for (int a : nums) {
      int l = a, r = a;
      auto itl = left.find(a - 1);
      if (itl != left.end()) l = itl->second;
      auto itr = right.find(a + 1);
      if (itr != right.end()) r = itr->second;

      updateMax(right, l, r);
      updateMin(left, r, l);

      ans = max(ans, r - l + 1);
    }
    return ans;
  }
};
```
