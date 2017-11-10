---
title: LeetCode 刷题记录 （七）
date: 2017-10-18 20:57:18
tags:
    - LeetCode
---

# Binary Tree Maximum Path Sum

> https://leetcode.com/problems/binary-tree-maximum-path-sum/description/

## 题意

给定一棵二叉树，二叉树上每个节点有一个整数权值。求在这棵二叉树上经过的节点的权值和最长的路径的权值和大小（路径至少经过一个节点）。

<!-- more -->

## 思路

对于一棵二叉树，上面的所有路径其实可以分为以下几种：

1. 只经过左子树中的节点
2. 只经过右子树中的节点
3. 只经过左子树中的节点和根
4. 只经过右子树中的节点和根
5. 左右子树节点和根都经过

很显然，第五种路径其实是第三、四种路径和根节点的组合；另外几种路径也和对应的子树的子树的某几种路径相关。

于是就可以递归计算，对每个节点分别计算在相应子树中五种路径的最大权值和即可。

## 代码

LeetCode直接提供了`TreeNode`的定义：

```c++
/**
* Definition for a binary tree node.
* struct TreeNode {
*     int val;
*     TreeNode *left;
*     TreeNode *right;
*     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
* };
*/
```

但上面挂载的信息还太少，又不能修改这个定义，于是只能新创建一个结构`SolNode`，添加一些成员，通过`TreeNode`进行构造。

因为`SolNode`在构造的时候会递归地对左右子树进行构造，正好和解题的顺序一样，所以干脆把解题过程也写在`SolNode`的构造函数里了。

```c++
struct SolNode {
    int val, leftMax, rightMax, midMax, ansMax;
    bool allNeg;
    int maxVal;
    SolNode *left, *right;
    SolNode(TreeNode *node) {
        if (!node) {
            ansMax = midMax = leftMax = rightMax = 0;
            allNeg = true;
            maxVal = -2e9;
            return;
        }
        left = new SolNode(node->left);
        right = new SolNode(node->right);
        val = node->val;
        leftMax = val + max(max(left->leftMax, left->rightMax), 0);
        rightMax = val + max(max(right->leftMax, right->rightMax), 0);
        midMax = leftMax + rightMax - val;
        int t1 = max(left->ansMax, right->ansMax);
        int t2 = max(leftMax, rightMax);
        ansMax = max(max(t1, t2), midMax);
        maxVal = max(max(left->maxVal, right->maxVal), val);
        allNeg = left->allNeg && right->allNeg && val < 0;
    }
};
class Solution {
public:
    int maxPathSum(TreeNode* root) {
        auto solNode = new SolNode(root);
        return solNode->allNeg ? solNode->maxVal : solNode->ansMax;
    }
};
```



# Minimum Window Substring

> https://leetcode.com/problems/minimum-window-substring/description/

## 题意

给定两个字符串$S$和$T$（题目没说，假设由*ascii*从0-255的字符都有可能出现），求$S$的最短子串$S'$，使得$T$中的每个字符在$S'$中都出现。

Node: 如果字符$c$在$T$中出现多次，那么$c$在$S'$中也应出现至少相同次数。

输入保证这样的最短子串是唯一的。

## 思路

~~其实题目标题中的**Window**就充满了暗示......~~

用两个迭代器$i$与$pos$分别为指向满足要求的子串的头与尾（以下标0方向为尾）。

在初始时$pos=0$，$i$不断增大直到$S[0:i]$满足要求为止，此时$S[0:i]$是一个满足要求的子串（不一定最短）。然后$i$每次右移一位，记$i^{'}=i+1$，此时得到的新子串$S[0:i^{'}]$也必定满足要求，但由于这个子串的左端可能会有多余的、删掉也不妨碍子串满足要求的字符，于是一直右移$pos$直到没有多余字符为止。

具体实现见下面代码。

## 代码

```c++
class Solution {
    int req[256], meet[256];
public:
    string minWindow(string s, string t) {
        if (s.length() < t.length()) return "";
        memset(req, 0, 256);
        memset(meet, 0, 256);
        int unMeet = 0;
        for (char c : t) {
            if (!req[c]) unMeet++;
            req[c]++;
        }
        int minLen = 2e9, minPos;
        bool ok = false;
        for (int i = 0, pos = 0; i < s.length(); i++) {
            meet[s[i]]++;
            if (meet[s[i]] == req[s[i]]) {
                // just meet
                unMeet--;
                if (!unMeet) ok = true;
            }
            while (meet[s[pos]] > req[s[pos]]) meet[s[pos++]]--;
            if (ok && i - pos + 1 < minLen) {
                minLen = i - pos + 1;
                minPos = pos;
            }
        }

        return ok ? s.substr(minPos, minLen) : "";
    }
};
```
