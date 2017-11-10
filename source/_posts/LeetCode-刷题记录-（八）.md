---
title: LeetCode 刷题记录 （八）
date: 2017-10-25 14:25:29
tags:
    - LeetCode
---

# Flatten Nested List Iterator

> https://leetcode.com/problems/flatten-nested-list-iterator/description/

## 题意

现在有一种新的嵌套数组类型，定义如下：

* `NestedInteger`可以表示一个整数，也可以表示为一个`NestedInteger`的数组。

现在要实现一个`NestedIterator`类，作为`NestedInteger`表示的嵌套数组的迭代器，它能够以该嵌套数组的“压平”后进行从左到右的迭代。

比如一个嵌套数组`[1, [2, 3], 4, [5, [6]]]`，迭代顺序就是`1, 2, 3, 4, 5, 6`。

<!-- more -->

## 思路

个人觉得这道题挺有意思的，比起算法，这题更像是一个语言题。

在一些动态类型的语言中（比如Python和JavaScript），这个问题经常遇到也比较容易解决。但在C++里面就比较难写了。

从嵌套数组的结构来分析，一个嵌套数组其实可以表示成一棵树，其中最底层的叶子节点就是各个整数元素，而非叶节点就是数组元素（空数组直接删掉不参与树的构建）。那么把这个嵌套数组压平的过程其实就是DFS的遍历过程。

但这道题所求的并不止是压平数组的结果，它还要求实现一个迭代器。也就是说这样一个DFS的遍历过程不是一口气完成的，而是每次调用next()方法都向前走一步。这也就要求有一个栈结构来保存执行上下文，手动模拟递归过程。

具体实现过程见代码。

## 代码

```c++
/**
 * // This is the interface that allows for creating nested lists.
 * // You should not implement it, or speculate about its implementation
 * class NestedInteger {
 *   public:
 *     // Return true if this NestedInteger holds a single integer, rather than a nested list.
 *     bool isInteger() const;
 *
 *     // Return the single integer that this NestedInteger holds, if it holds a single integer
 *     // The result is undefined if this NestedInteger holds a nested list
 *     int getInteger() const;
 *
 *     // Return the nested list that this NestedInteger holds, if it holds a nested list
 *     // The result is undefined if this NestedInteger holds a single integer
 *     const vector<NestedInteger> &getList() const;
 * };
 */
class NestedIterator {
    typedef pair<const vector<NestedInteger>*, vector<NestedInteger>::const_iterator> Context; // second must be integer
    stack<Context> stk;

    void decompose(const NestedInteger &ni) {
        auto p = &ni;
        while (!p->isInteger()) {
            auto& list = p->getList();
            if (list.size()) {
                // ni not an empty list
                auto context = make_pair(&list, list.begin());
                stk.push(context);
                p = &*list.begin();
            }
            else break;
        }
    }

    int getCurrent() {
        skipNull();
        auto context = stk.top();
        return context.second->getInteger();
    }

    bool isEmptyList(const NestedInteger &ni) {
        return !ni.isInteger() && ni.getList().empty();
    }

    void moveNext() {
        while (true) {
            stk.top().second++;
            if (stk.top().second == stk.top().first->end()) {
                stk.pop();
                if (stk.empty()) break;
            }
            else break;
        }
        if (!stk.empty()) decompose(*stk.top().second);
    }

    void skipNull() {
        while (!stk.empty() && isEmptyList(*stk.top().second)) moveNext();
    }

public:
    NestedIterator(vector<NestedInteger> &nestedList) {
        if (nestedList.empty()) return;
        stk.push(make_pair(&nestedList, nestedList.cbegin()));
        decompose(nestedList.front());
    }

    int next() {
        int result = getCurrent();

        moveNext();

        return result;
    }

    bool hasNext() {
        skipNull();
        return !stk.empty();
    }
};

/**
 * Your NestedIterator object will be instantiated and called as such:
 * NestedIterator i(nestedList);
 * while (i.hasNext()) cout << i.next();
 */
```
