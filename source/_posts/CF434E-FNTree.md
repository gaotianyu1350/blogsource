title: "[Solution][CF434E]Furukawa Nagisa's Tree"
date: 2015-03-06 16:51:42
tags: [CF,点分治,补集思想]
categories: 题解
---
## 题目描述
[传送门](http://codeforces.com/problemset/problem/434/E)

## 分析
显然题目要求一个小于$O(n^2)$的算法，直接计算满足条件的三元组个数似乎很棘手。关于三元组的问题常常用`补集思想`，那我们可以先计算不满足条件的三元组的个数。先假设如果两个点之间的距离模$Y$等于$X$，就为一条黑边，否则为白边 。一个三元环要么颜色全部一样（满足条件），要么有一条边颜色不一样（不满足条件）。令$in0_i$表示从$i$出发的边有多少是黑边，$in1_i$表示到$i$的有多少条边是黑边。$out0_i$和$out1_i$同理。那么不满足条件的三元组的个数为两倍：
$$ \sum 2in0_iin1_i+2out0_iout1_i+in0_iout1_i+in1_iout0_i$$

为什么是两倍？因为对于一个不满足条件的三元组，有两个点都是黑白相间的边。

如何求这四个数组？用点分治可以解决。用两个`map`记录剩余系中各个数对应的个数（出和入），最后别忘了特殊处理根的情况，并减去同一子树重复统计的情况，就可以了。

不要在意常数……

<!--more-->
## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <map>
#include <algorithm>
using namespace std;

typedef long long ll;
const int MaxN = 1e5 + 100;
const int MaxM = 2e5 + 100;
const int inf = 1e9;

int point[MaxN], nxt[MaxM], v[MaxM], tot = 0;
int n, k;
ll value[MaxN], X, Y, inv[MaxN], pw[MaxN];
ll in[MaxN][2], out[MaxN][2];
bool cut[MaxN];

int f[MaxN], fa[MaxN], size[MaxN], cur[MaxN], cntIn, cntOut;
map<int, int> inDy, outDy;

inline void addedge(int x, int y) {
	tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y;
	tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x;
}

inline int getnum() {
	int ans = 0; char c; bool flag = false;
	while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
	if (c == '-') flag = true; else ans = c - '0';
	while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
	return ans * (flag ? -1 : 1);
}

inline ll qE(ll a, int p, ll mod) {
	ll ans = 1;
	for (; p; p >>= 1, a = a * a % mod)
		if (p & 1)
			ans = ans * a % mod;
	return ans;
}

inline void init() {
	for (int i = 1; i <= n; i++) {
		pw[i] = qE(k, i, Y);
		inv[i] = qE(pw[i], Y - 2, Y);
	}
}

inline int query(map<int, int> &Map, int x) {
	map<int, int>::iterator p = Map.find(x);
	if (p != Map.end())
		return (*p).second;
	else
		return 0;
}

inline void add(map<int, int> &Map, int x, int v) {
	map<int, int>::iterator p = Map.find(x);
	if (p != Map.end())
		(*p).second += v;
	else
		Map[x] = v;
}

inline int calcSize(int now, int fa) {
	int size = 1;
	for (int tmp = point[now], u; u = v[tmp], tmp; tmp = nxt[tmp])
		if (u != fa && !cut[u])
			size += calcSize(u, now);
	return size;
}

inline void calcRoot(int now, int fa, int totNode, int &minn, int &wh) {
	size[now] = 1;
	f[now] = 0;
	
	for (int tmp = point[now], u; u = v[tmp], tmp; tmp = nxt[tmp])
		if (u != fa && !cut[u]) {
			calcRoot(u, now, totNode, minn, wh);
			size[now] += size[u];
			f[now] = max(f[now], size[u]);
		}
	f[now] = max(f[now], totNode - size[now]);
	if (f[now] < minn)
		minn = f[now], wh = now;
}

inline void calcOut(int now, ll lastRm, int fa, int deep, int delta) {
	ll remain = (lastRm + value[now] * pw[deep]) % Y;
	add(outDy, remain, delta); cntOut += delta;
	
	for (int tmp = point[now], u; u = v[tmp], tmp; tmp = nxt[tmp])
		if (u != fa && !cut[u])
			calcOut(u, remain, now, deep + 1, delta);
}

// A * pw[deep] + tmp = X
// A * pw[deep] = X + Y - tmp
// A = (X + Y - tmp) * inv[deep]
inline void countOut(int now, ll lastRm, int fa, int deep, int delta) {
	ll remain = (lastRm * k + value[now]) % Y;
	ll A = (X + Y - remain) % Y * inv[deep] % Y;
	ll res = query(outDy, A);
	out[now][0] += res;
	out[now][1] += cntOut - res;
	add(inDy, A, delta); cntIn += delta;
	
	for (int tmp = point[now], u; u = v[tmp], tmp; tmp = nxt[tmp])
		if (u != fa && !cut[u])
			countOut(u, remain, now, deep + 1, delta);
}

inline void countIn(int now, ll lastRm, int fa, int deep, int delta) {
	ll remain = (lastRm + value[now] * pw[deep]) % Y;
	ll res = query(inDy, remain);
	in[now][0] += res;
	in[now][1] += cntIn - res;
	
	for (int tmp = point[now], u; u = v[tmp], tmp; tmp = nxt[tmp])
		if (u != fa && !cut[u])
			countIn(u, remain, now, deep + 1, delta);
}

inline void solve(int node) {
	int minn = inf, root = 0, totNode = calcSize(node, 0);
	calcRoot(node, 0, totNode, minn, root);
	cut[root] = true;
	for (int tmp = point[root], u; u = v[tmp], tmp; tmp = nxt[tmp])
		if (!cut[u])
			solve(u);
	
	inDy.clear(); outDy.clear(); cntIn = 0; cntOut = 0;
	
	add(outDy, value[root], 1); cntOut++;
	for (int tmp = point[root], u; u = v[tmp], tmp; tmp = nxt[tmp])
		if (!cut[u]) calcOut(u, value[root], 0, 1, 1);
	
	ll res = query(outDy, X);
	out[root][0] += res;
	out[root][1] += cntOut - res;
	add(inDy, X, 1); cntIn++;
	for (int tmp = point[root], u; u = v[tmp], tmp; tmp = nxt[tmp])
		if (!cut[u]) countOut(u, 0, 0, 1, 1);

	res = query(inDy, value[root]);
	in[root][0] += res;
	in[root][1] += cntIn - res;	
	for (int tmp = point[root], u; u = v[tmp], tmp; tmp = nxt[tmp])
		if (!cut[u]) countIn(u, value[root], 0, 1, 1);
		
	for (int tmp = point[root], u; u = v[tmp], tmp; tmp = nxt[tmp])
		if (!cut[u]) {
			inDy.clear(); outDy.clear(); cntIn = 0; cntOut = 0;
			calcOut(u, value[root], 0, 1, -1);
			countOut(u, 0, 0, 1, -1);
			countIn(u, value[root], 0, 1, -1);
		}
	cut[root] = false;
}

int main() {
	n = getnum(); Y = getnum(); k = getnum(); X = getnum();
	X %= Y; 
	for (int i = 1; i <= n; i++) value[i] = getnum() % Y;
	for (int i = 1; i < n; i++) {
		int x, y;
		x = getnum(); y = getnum();
		addedge(x, y);
	}
	
	init();
	solve(1);
	
	ll ans = (ll)n * n * n * 2;
	for (int i = 1; i <= n; i++)
		ans -= 2 * in[i][0] * in[i][1] + 2 * out[i][0] * out[i][1] + in[i][0] * out[i][1] + in[i][1] * out[i][0];
	ans /= 2;
	
	cout << ans << endl;
}
```