title: "[Solution][BZOJ3550][ONTAK2010]Vacation"
date: 2015-03-23 14:48:50
tags: [BZOJ,ONTAK,费用流]
categories:
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3550)

## 分析
费用流？DP？线性规划？

线性规划太神看不懂，DP太费脑子懒得想，于是就费用流吧……

<!--more-->

首先上图：

![BZOJ3550](/img/BZOJ3550db.png)

其实看到这个图就能明白个大概了。我们首先把整个序列分成三部分，每个部分有$n$个元素。由于每相邻的$n$个元素里面只能选择$k$个元素，我们就通过一个虚拟源点限制总的流量不超过$k$。然后再从虚拟源点向前$n$个点连容量为$1$，费用为其价值的边。流它就表示选这个点。这样第一部分的限制就保证了。

显然这些流还要继续往下流。从虚拟源点向$1$连无穷大的边，然后从$1$往后每两个点之间连无穷大的边，表示忽略这些点往下走。这样显然是合法的。然后从点$i$（$1\leq i\leq n$）向$i+n$连容量$1$，费用为$i+n$权值的边，从$i+n$向$i+2n$连容量$1$，费用$i+2n$权值的边。这样的话如果选择了$x$点（如图中红色标注），然后选择了$x+n$点，一定能够$(x+1,x+n)$区间的合法。

其实这个模型也就是线性规划的模型，只不过用了个费用流的方式去理解而已。

## 代码
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <queue>
#include <algorithm>
using namespace std;

const int MaxN = 220;
const int MaxNode = 500;
const int MaxM = 2 * (600 + 400 +100);
const int inf = 2e9;

int point[MaxNode], nxt[MaxM], v[MaxM], remain[MaxM], w[MaxM], tot;
int dis[MaxNode], lastedge[MaxNode];
bool inq[MaxNode];

inline void addedge(int x, int y, int cap, int cost) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y;
    remain[tot] = cap; w[tot] = cost;
    tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x;
    remain[tot] = 0; w[tot] = -cost;
}

inline void init() {
    tot = -1;
    memset(point, -1, sizeof(point));
    memset(nxt, -1, sizeof(nxt));
}

inline int addflow(int s, int t) {
    int ans = inf, now = t;
    while (now != s) {
        ans = min(ans, remain[lastedge[now]]);
        now = v[lastedge[now] ^ 1];
    }
    now = t;
    while (now != s) {
        remain[lastedge[now]] -= ans;
        remain[lastedge[now] ^ 1] += ans;
        now = v[lastedge[now] ^ 1];
    }
    return ans;
}

inline bool spfa(int s, int t, int &maxflow, int &mincost) {
    memset(inq, 0, sizeof(inq));
    memset(dis, 0x7f, sizeof(dis));
    queue<int> q;
    dis[s] = 0; inq[s] = true; q.push(s);

    while (!q.empty()) {
        int now = q.front(); q.pop(); inq[now] = false;
        for (int tmp = point[now], u; u = v[tmp], tmp != -1; tmp = nxt[tmp])
            if (remain[tmp] && dis[u] > dis[now] + w[tmp]) {
                dis[u] = dis[now] + w[tmp];
                lastedge[u] = tmp;
                if (!inq[u])
                    inq[u] = true, q.push(u);
            }
    }
    if (dis[t] > inf) return false;
    int add = addflow(s, t);
    maxflow += add; mincost += add * dis[t];
    return true;
}

inline void mfmc(int s, int t, int &maxflow, int &mincost) {
    maxflow = mincost = 0;
    while (spfa(s, t, maxflow, mincost));
}

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

int n, k, value[MaxN * 3];

int main() {
    init();
    n = getnum(); k = getnum();
    for (int i = 1; i <= n * 3; i++)
        value[i] = getnum();   

    int s = n * 2 + 1, vs = n * 2 + 2, t = n * 2 + 3;
    addedge(s, vs, k, 0);
    addedge(vs, 1, inf, 0);
    addedge(2 * n, t, inf, 0);
    for (int i = 1; i < n * 2; i++)
        addedge(i, i + 1, inf, 0);
    for (int i = 1; i <= n; i++) {
        addedge(vs, i, 1, -value[i]);
        addedge(i, i + n, 1, -value[i + n]);
        addedge(i + n, t, 1, -value[i + 2 * n]);
    }

    int maxflow, mincost;
    mfmc(s, t, maxflow, mincost);
    printf("%d\n", -mincost);
}
```