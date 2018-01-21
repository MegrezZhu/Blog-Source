---
title: LeetCode 刷题记录 （十九）
date: 2018-01-05 23:29:25
tags:
	- LeetCode
---

# Number of Atoms

> https://leetcode.com/problems/number-of-atoms/description/

## 题意

给定一个字符串形式的化学式（如$H_2O$），计算里面各种原子的个数，并按元素符号升序输出。

<!-- More -->

## 思路

题意很简单，但因为化学式中也有括号嵌套（如$Mg(OH)_2$），因此不能直接无脑扫一遍得结果，需要用个栈来处理括号。

其余都是一些细节上的问题，具体实现见下面代码。

## 代码

```c++
class Solution {
	typedef map<string, int> Data;

	enum STATE {
		
	};

	void combine(Data &a, const Data &b, int multiplier = 1) {
		for (const auto &p : b) {
			auto it = a.find(p.first);
			if (it == a.end()) it = a.insert(make_pair(p.first, 0)).first;
			it->second += p.second * multiplier;
		}
	}

	void fold(stack<Data> &stk, int multiplier = 1) {
		auto top = move(stk.top());
		stk.pop();
		combine(stk.top(), top, multiplier);
	}
public:
	string countOfAtoms(string formula) {
		stack<Data> stk;
		string buf;
		stk.push(Data());

		for (size_t pos = 0; pos < formula.length(); pos++) {
			char ch = formula[pos], peek = pos + 1 != formula.length() ? formula[pos + 1] : '\0';

			if (ch == '(') {
				stk.push(Data());
				continue;
			}
			if (ch == ')') {} // nothing
			if (isdigit(ch)) {
				buf += ch;
				if (!isdigit(peek)) {
					fold(stk, stoi(buf));
					buf.clear();
				}
				continue;
			}
			if (isupper(ch)) {
				buf = ch;
				if (!islower(peek) && !isdigit(peek)) {
					stk.push(Data({ {buf, 1} }));
					fold(stk);
					buf.clear();
				}
				else if (isdigit(peek)) {
					stk.push(Data({ { buf, 1 } }));
					buf.clear();
				}
				continue;
			}
			if (islower(ch)) {
				buf += ch;
				if (!islower(peek) && !isdigit(peek)) {
					stk.push(Data({ { buf, 1 } }));
					fold(stk);
					buf.clear();
				}
				else {
					stk.push(Data({ { buf, 1 } }));
					buf.clear();
				}
				continue;
			}
		}

		string res;
		for (auto &p : stk.top()) {
			res += p.first;
			if (p.second > 1) res += to_string(p.second);
		}
		return res;
	}
};
```

