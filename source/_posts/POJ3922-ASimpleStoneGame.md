title: "[Solution][POJ3922]A simple stone game"
date: 2015-04-06 11:32:54
tags: [POJ,博弈论]
categories: 题解
---
## 题目描述

## 分析
博弈什么的最蛋疼了……

通过打表可以发现，当$k=1$的时候，无解状态为$2$的整数次幂，当$k=2$的时候，无解状态为斐波那契数列，很有意思对不对……

当$k=1$时，如果现在不是$2$的整数次幂，那么我们可以取最低位的那个二进制位，由于对手不能取的比我多，他必定只能取一个二进制为$0$的位，这就会制造一些新的$1$，这样对手总不能取完。

当$k=2$的时候，斐波那契数列有个性质是可以用不相邻的一些元素的和来表示任何正整数。不相邻就意味着前项的二倍小于后一项。仿照$k=1$时的情况，我们每次取最小的那个斐波那契数，对手显然就只能拆散下一个数，永远取不完。

对于$k>2$的情况，我们可以自己构造一个类似的数列。令$a_i$为构造的数列，$b_i$为前$i$个数能够加出来的和。显然$a_{i+1}=b_i+1$，$b_{i+1}=b_j+a_{i+1}$，其中$j$为最后一个满足$a_{i+1}>ka_j$的数字的编号。

最后，如果$n$为某个$a$，则输，否则，用$a$构造出$n$，最小的一个就是答案。

<!--more-->
## 代码
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <vector>
using namespace std;

typedef long long ll;
const int MaxN = 1e7;
const int INF = 1e9;
ll a[MaxN], b[MaxN];

inline ll getnum() {
	ll ans = 0; char c; bool flag = false;
	while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
	if (c == '-') flag = true; else ans = c - '0';
	while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
	return ans * (flag ? -1 : 1);
}

int main() {
	int testcase = getnum();
	for (int t = 1; t <= testcase; t++) {
		int top = 0;
		ll n = getnum();
		ll k = getnum();
		
		printf("Case %d: ", t);
		
		bool isok = true; int j = 0;
		while (b[top] < n) {
			top++;
			a[top] = b[top - 1] + 1;
			
			while (a[j] * k < a[top]) j++;
			b[top] = b[j - 1] + a[top];
			
			if (a[top] == n) {
				printf("lose\n");
				isok = false;
				break;
			}
		}
		
		if (!isok) continue;
		int last = INF;
		for (int i = top; i > 0; i--) {
			if (a[i] <= n)
				last = a[i], n -= last;
			if (!n) break;
		}
		printf("%d\n", last);
	}
}
```