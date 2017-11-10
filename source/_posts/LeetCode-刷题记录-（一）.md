---
title: LeetCode 刷题记录 （一）
date: 2017-09-08 15:20:41
tags:
  - LeetCode
---

# merge-k-sorted-lists

> https://leetcode.com/problems/merge-k-sorted-lists/description/

## 题意

> Merge *k* sorted linked lists and return it as one sorted list. Analyze and describe its complexity.

给定k个已排序（升序）的链表，将他们合成为一个大的已排序链表。

<!-- more -->

## 解题思路

当`k=2`时就是一个典型的`Merge Sort`的情况，在这时就只需要每次比较两个待归并链表的首项并把较小的那一项加到新的已排序链表的尾部即可。

扩展到k路归并的话，思路同样是取k个待归并列表的首项并将最小的一项加到结果链表的尾部。对于比较k个首项的大小并获得最小项的过程，我们可以维护一个由所有链表头组成的最小堆（直接用`algorithm`库的`make_heap` `push_heap` `pop_heap` 完成，省去自己实现），这样每次得到最小项所需的复杂度为`O(1)` ，添加删除堆节点的复杂度为`O(logK)` 。最终的复杂度为`O(NlogK)`，其中N为k个链表的总节点数。

## 坑点

会给空链表，即输入的vector中会有NULL项。

## 代码

```c++
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
#include <algorithm>
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        auto head = new ListNode(0), tail = head;
        vector<ListNode*> heap;
        for (auto it = lists.begin(); it != lists.end(); it++) {
            auto node = *it;
            if (node) heap.push_back(node);
        }
        auto comp = [](ListNode* a, ListNode* b) {
            return a->val > b->val;
        };
        make_heap(heap.begin(), heap.end(), comp);
        while (heap.size()) {
            pop_heap(heap.begin(), heap.end(), comp);
            ListNode* selected = heap.back();
            heap.pop_back();
            tail = tail->next = new ListNode(selected->val);
            if (selected->next) {
                heap.push_back(selected->next);
                push_heap(heap.begin(), heap.end(), comp);
            }
        }
        return head->next;
    }
};
```
