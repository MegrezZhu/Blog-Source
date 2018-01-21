---
title: LeetCode 刷题记录（十四）
date: 2017-12-24 12:41:05
tags:
    - LeetCode
---


# Find Median from Data Stream

> https://leetcode.com/problems/find-median-from-data-stream/description/

## 题意

要求实现一个容器类，能够：

* 将一个元素加入容器中
* 计算当前容器中所有元素的中位数

<!-- More -->

## 思路

维护两个堆，大根堆$H_1$和小根堆$H_2$，$H_1$里面是当前容器中最小的$\lfloor\frac{N+1}{2}\rfloor$个元素，相应的$H_2$里面是当前容器中最大的$N-\lfloor\frac{N+1}{2}\rfloor$个元素。在有新元素加入时处理一下新元素和两个堆的根元素之间的移动关系即可，具体处理逻辑见下面代码。

在$N$为奇数时，中位数就是$H_1$的根；否则中位数是两个堆的根的均值。

时间复杂度：添加元素$\mathcal{O}(\log N)$，计算当前中位数$\mathcal{O}(1)$。

## 代码

```c++
class MedianFinder {
    vector<int> left, right;
    int size;
public:
    /** initialize your data structure here. */
    MedianFinder() {
        size = 0;
    }

    void addNum(int num) {
        size++;
        int leftCap = (size + 1) / 2, rightCap = size - leftCap;
        if (left.size() < leftCap) {
            int toLeft;
            if (right.size() == 0 || num <= right.front()) {
                toLeft = num;
            } else {
                toLeft = right.front();
                pop_heap(right.begin(), right.end(), greater<int>());
                right.pop_back();
                right.push_back(num);
                push_heap(right.begin(), right.end(), greater<int>());
            }
            left.push_back(toLeft);
            push_heap(left.begin(), left.end());
        }
        else {
            int toRight;
            if (left.size() == 0 || num >= left.front()) {
                toRight = num;
            }
            else {
                toRight = left.front();
                pop_heap(left.begin(), left.end());
                left.pop_back();
                left.push_back(num);
                push_heap(left.begin(), left.end());
            }
            right.push_back(toRight);
            push_heap(right.begin(), right.end(), greater<int>());
        }
    }

    double findMedian() const {
        if (size % 2) return left.front();
        else return (left.front() + right.front()) / 2.0;
    }
};
```
