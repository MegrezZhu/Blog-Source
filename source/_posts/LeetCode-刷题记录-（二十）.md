---
title: LeetCode 刷题记录 （二十）
date: 2018-01-10 19:36:40
tags:
	- LeetCode
---

# Find Largest Value in Each Tree Row

> https://leetcode.com/problems/find-largest-value-in-each-tree-row/description/

## 题意

给定一个二叉树，对二叉树的每一行（深度相同的所有节点视为在同一行），找出行内的最大元素。

## 思路

BFS。

<!-- More -->

## 代码

```c++
class Solution {
public:
    vector<int> largestValues(TreeNode* root) {
        if (!root) return {};
        vector<int> res;
        list<TreeNode*> parent = { root };
        while (!parent.empty()) {
            list<TreeNode*> son;
            int _max = -2147483648;
            while (!parent.empty()) {
                auto p = parent.front();
                parent.pop_front();
                if (p->left) son.push_back(p->left);
                if (p->right) son.push_back(p->right);
                _max = max(_max, p->val);
            }
            res.push_back(_max);
            parent = move(son);
        }
        
        return res;
    }
};
```

# Partition Equal Subset Sum

> https://leetcode.com/problems/partition-equal-subset-sum/description/

## 题意

对于一个正整数数组$A$，判断这个数组是否可以被划分成两个和相等的子数组（不需要连续）。

## 思路

经典背包题。考虑到两个子数组的元素和相等，显然对于每个子数组，其和为$\frac{\sum_{i}A_i}{2}$。于是可以转换一个等价的0-1背包问题：背包容量为$\frac{\sum_{i}A_i}{2}$，是否能装满？

## 代码

```c++
class Solution {
public:
  bool canPartition(vector<int>& nums) {
    int sum = 0;
    for (int x : nums) sum += x;
    if (sum % 2) return false;
    vector<bool> f(sum / 2 + 1);
    f[0] = true;
    for (int i = 1; i < nums.size(); i++) {
      for (int j = sum / 2; j >= 0; j--) {
        if (j - nums[i] >= 0) f[j] = f[j] || f[j - nums[i]];
      }
    }
    return f[sum / 2];
  }
};

```

