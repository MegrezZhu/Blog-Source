---
title: LeetCode 刷题记录 （十一）
date: 2017-11-15 15:25:37
tags:
  - LeetCode
---

# Longest Valid Parentheses

> https://leetcode.com/problems/longest-valid-parentheses/description/

## 题意

给出一串只包含`(`与`)`的括号字符串$S$，找出里面最长的合法括号子串的长度。

<!-- more -->

## 思路

回忆普通的括号匹配合法性检测，做法都是用一个栈结构维护目前待匹配的括号字符，然后每次取栈顶的字符与当前即将添加的字符比较，如果匹配则栈顶元素退栈，最后如果栈为空的话，则为合法的括号序列。

对比这道题，可以发现一个很有用的特性：

在上述括号匹配检测的过程中，在处理完第$i$个字符$S_i$后，若栈顶元素为$S_j$（显然$j<i$），那么$S_{j+1}, S_{j+2}, ..., S_i$都不在栈中，也就是说这是一段合法的括号匹配序列。显然这也是以$S_i$为结尾的最长合法括号匹配序列。

那么这个问题其实就很简单了，只需要进行一次naive的括号匹配过程，并在每次处理完一个字符$S_i$后根据栈顶元素的在$S$中的位置得到以$S_i$结尾的最长合法括号匹配序列的长度并更新答案即可。

## 代码

```c++
class Solution {
  bool match(char a, char b) {
    return a == '(' && b == ')';
  }
public:
  int longestValidParentheses(string s) {
    stack<pair<char, int>> stk;
    int ans = 0;
    for (int i = 0; i < s.length(); i++) {
      char c = s[i];
      if (stk.empty() || !match(stk.top().first, c)) stk.push(make_pair(c, i));
      else {
        stk.pop();
        int left = stk.empty() ? -1 : stk.top().second;
        ans = max(ans, i - left);
      }
    }
    return ans;
  }
};
```
