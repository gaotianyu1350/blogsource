title: "[Solution][BZOJ2806][CTSC2012]Cheat"
date: 2015-04-08 19:21:54
tags: [BZOJ,CTSC,SAM,DP,单调队列]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=2555)

## 分析
`SAM`+`DP`+单调队列

<!--more-->
## 代码
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <queue>
using namespace std;

const int MaxLen = 1.1 * 1e6 + 100;
const int MaxDBLen = 1e5 + 100;

namespace sam {
	struct sam_node {
		int val;
		sam_node *pre, *ch[2];
		sam_node() {
			val = 0; 
			pre = ch[0] = ch[1] = 0;
		}
	} *root, *last, pool[MaxLen];
	int cnt = 0;
	
	inline void init() {
		root = last = pool;
	}
	
	inline void reset() {
		last = root;
	}
	
	void add(int x) {
		sam_node *p = last;
		sam_node *np = &pool[++cnt]; np->val = p->val;
		while (p && !p->ch[x])
			p->ch[x] = np, p = p->pre;
		if (!p)
			np->pre = root;
		else {
			sam_node *q = p->ch[x];
			if (q->val == p->val + 1)
				np->pre = q;
			else {
				sam_node *nq = &pool[++cnt]; nq->val = p->val + 1;
				nq->pre = q->pre;
				memcpy(nq->ch, q->ch, sizeof(q->ch));
				np->pre = q->pre = nq;
				while (p && p->ch[x] == q)
					p->ch[x] = nq, p = p->pre;
			}
		}
		last = np;
	}
}

namespace qu {
	int q[MaxDBLen], loc[MaxDBLen];
	int head, tail;
	
	inline void init() {
		head = tail = 1;
	}
	
	inline int getbest(int last) {
		while (head < tail && loc[head] < last)
			head++;
		return q[head];
	}
	
	inline void add(int value, int l) {
		while (head < tail && q[tail - 1] < value)
			tail--;
		q[tail++] = value;
		loc[tail - 1] = l;
	}
}

char s[MaxDBLen];
int f[MaxDBLen];
int n, m;

inline void getinit() {
	sam::init();
	char c;
	
	scanf("%d%d", &n, &m);
	getchar();
	int curidx = 1;
	
	while (curidx <= m) {
		c = getchar();
		if (c != '0' && c != '1') {
			curidx++;
			sam::reset();
		}
		else {
			sam::add(c - '0');
		}
	}
}

inline void getstring(char *s, int &len) {
	len = 0;
	char c; 
	while ((c = getchar()) == '0' || c == '1')
		s[len++] = c;
}

inline bool solve(int len, int L, int correct) {
	sam::sam_node *cur = sam::root;
	qu::init();
	int pplen = 0, last = 0, lastmax = 0;
	
	f[0] = 0;
	for (int i = 1; i <= len; i++) {
		int x = s[i - 1] - '0';
		// PiPei
		if (cur->ch[x]) cur = cur->ch[x], pplen++;
		else {
			while (cur && !cur->ch[x])
				cur = cur->pre;
			if (!cur) cur = sam::root, pplen = 0;
			else pplen = cur->val + 1, cur = cur->ch[x]; 
		}
		
		while (last < i - pplen) {
			last++;
			lastmax = max(lastmax, f[last]);
		}
		
		// DP
		f[i] = f[i - 1];
		if (pplen >= L) {
			f[i] = max(f[i], lastmax + pplen);
			f[i] = max(f[i], qu::getbest(i - pplen) - (len - i));
		}
		
		if (i - L + 1 >= 0) {
			int idx = i - L + 1;
			qu::add(f[idx] + len - idx, idx);
		}
	}
	
	if (f[len] >= correct)
		return true;
	else
		return false;
}

inline void check() {
	int tar = sizeof(s) + sizeof(f) + sizeof(qu::q) + sizeof(qu::loc) + sizeof(sam::pool);
	printf("%d\n", tar / 1000000);
}

int main() {
	getinit();
	
	for (int t = 1; t <= n; t++) {
		int len;
		getstring(s, len);
		
		int correct = ceil(0.9 * len);
		
		int L = 0, R = len;
		while (L < R) {
			int mid = (L + R + 1) >> 1;
			if (solve(len, mid, correct))
				L = mid;
			else
				R = mid - 1;
		}
		
		printf("%d\n", L);
	}
}

```