---
title: 对STINGY SAT问题属于NPC的简单证明
date: 2018-01-21 21:26:15
tags:
	- NP
---

> STINGY SAT is the following problem: given a set of clauses (each a disjunction of literals) and an integer k, find a satisfying assignment in which at most k variables are true, if such an assignment exists.

简单来说，*STINGY SAT*（该翻译成*贪婪SAT*？）问题是*SAT*（Boolean Satisfiability Problem）的扩展，增加了一个限制：解中True的数量不能超过k个。

<!-- More -->

---

根据证明*NPC*（NP-Complete）问题的基本做法，主要需要找出从一个NPC问题规约到*STINGY SAT*问题的方法，在这里我直接选择了*SAT进行规约*。

规约的方式是显而易见的：对于有$n$个变量的*SAT*的问题，其解中True的个数必然不超过$n$个，因此一个有$n$个变量的*SAT*可以规约成一个$k=n$的*STINGY SAT*问题。

进行规约之后，根据如下证明：

* 若*STINGY SAT*有多项式解法，那么就可以用这个解法同样在多项式时间内解决*SAT*问题，于是*SAT*不是*NPC*。
* 于是与*SAT*是*NPC*的前提矛盾。
* 因此*STINGY SAT*也是*NPC*。

证毕。