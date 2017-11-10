---
title: LeetCode 刷题记录 （四）
date: 2017-09-27 14:30:54
tags:
    - LeetCode
---

# Sliding Window Maximum

> https://leetcode.com/problems/sliding-window-maximum/description/

## 题意

一个有`n`个数字的数组`a`，对于给定的`k`，对每一组连续的k个数组元素（即`a[i...i+k-1]` ），输出这k个元素中的最大值。

<!-- more -->

##    思路

注意到对于`a[i]`，如果存在`j > i`使得`a[j] >= a[i]`，那么对于所有末端位置大于等于`j`的k元组，`a[i]`都不可能会是这个k元组中的最大值，基于只可能出现以下几种情况：

* `a[i]`不在这个k元组中，此时`a[i]`自然不可能是这个k元组的最值；
* `a[i]`在这个k元组中，但因该k元组末端位置在`j`右侧，即`a[j]`也在k元组中，`a[i]`被`a[j]` 淘汰；

因此可以枚举k元组的右侧端点`t`，同时维护一个队列：每当`t`右移，则将`a[t]`加入队列尾，在加入前淘汰（移除）所有在队列中的不大于`a[t]`的元素。经过这一过程后的队列头就是以`t`为右侧端点的k元组的最大值。

## 代码

```c++
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        list<pair<int, int>> q;
        vector<int> res;

        for (int i = 0; i < nums.size(); i++) {
            while (!q.empty() && q.front().second < i - k + 1) q.pop_front();
            while (!q.empty() && q.back().first < nums[i]) q.pop_back();
            q.push_back({ nums[i], i });

            if (i >= k - 1) res.push_back(q.front().first);
        }

        return res;
    }
};
```



#    Binary Tree Postorder Traversal

> https://leetcode.com/problems/binary-tree-postorder-traversal/description/

## 题意

给定一个二叉树，以后序遍历方式输出所有的节点。（这题居然也是Hard）

##    代码

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *         int val;
 *         TreeNode *left;
 *         TreeNode *right;
 *         TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
        vector<int> res;
public:
        vector<int> postorderTraversal(TreeNode* root) {
                res.clear();
                traverse(root);
                return res;
        }

        void traverse(TreeNode* root) {
                if (!root) return;
                traverse(root->left);
                traverse(root->right);
                res.push_back(root->val);
        }
};
```
