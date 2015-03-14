title: "[Solution][BZOJ1520][POI2006]Szk-Schools"
date: 2014-12-27 15:42:11
tags: [BZOJ,POI,网络流,二分图最大完美匹配,费用流]
categories: 题解
---
刷水
<!--more-->
## 题目
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1520)

## 分析
二分图最大完美匹配问题。因为每个学校要对应唯一的编号，所以可以建二分图。学校和每一个它可以拥有的编号连边，容量任意，费用为它需要付出的代价。学校和编号分别向源、汇连容量为$1$、费用为$0$的边。跑最小费用最大流即可。（忘记`KM`怎么写了所以只好写费用流）

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
#include <queue>
using namespace std;
const int MAXN = 210;
const int MAXNODE = 500;
const int MAXM = 2 * 200 * 200 + 2 * 200 * 2 + 10;
const int inf = 1e9;
int point[MAXNODE], nxt[MAXM], w[MAXM], v[MAXM], remain[MAXM], tot;
int lastedge[MAXNODE], dis[MAXNODE];
bool inq[MAXNODE]; int n, node;
queue<int> q;
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
inline void init() {
    memset(point, -1, sizeof(point));
    memset(nxt, -1, sizeof(nxt)); tot = -1;
}
inline int addflow(int s, int t) {
    int ans = inf, now = t;
    while (now != s) {
        ans = min(ans, remain[lastedge[now]]);
        now = v[lastedge[now] ^ 1];
    } now = t;
    while (now != s) {
        remain[lastedge[now]] -= ans;
        remain[lastedge[now] ^ 1] += ans;
        now = v[lastedge[now] ^ 1];
    } return ans;
}
inline bool spfa(int s, int t, int &maxflow, int &mincost) {
    memset(dis, 0x7f, sizeof(dis)); dis[s] = 0;
    memset(inq, 0, sizeof(inq)); inq[s] = true;
    while (!q.empty()) q.pop(); q.push(s);
    while (!q.empty()) {
        int now = q.front(); q.pop(); inq[now] = false;
        for (int tmp = point[now]; tmp != -1; tmp = nxt[tmp]) 
            if (dis[now] + w[tmp] < dis[v[tmp]] && remain[tmp]) {
                dis[v[tmp]] = dis[now] + w[tmp]; lastedge[v[tmp]] = tmp;
                if (!inq[v[tmp]]) inq[v[tmp]] = true, q.push(v[tmp]);               
            }
    } if (dis[t] > inf) return false; int adf;
    maxflow += (adf = addflow(s, t)); mincost += adf * dis[t];
    return true;
}
inline void solve(int s, int t, int &maxflow, int &mincost) {
    maxflow = mincost = 0;
    while (spfa(s, t, maxflow, mincost));
}
inline void addedge(int x, int y, int cap, int ww) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y; remain[tot] = cap, w[tot] = ww;
    tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x; remain[tot] = 0; w[tot] = -ww;
}
int main() {
    init(); n = getnum();
    node = 2 * n + 2; int s = node - 1, t = node;
    for (int i = 1; i <= n; i++) {
        int m, a, b, k;
        m = getnum(); a = getnum(); b = getnum(); k = getnum();
        for (int j = a; j <= b; j++) addedge(i, n + j, 1, k * fabs(m - j));
        addedge(s, i, 1, 0); addedge(i + n, t, 1, 0);
    } int maxflow, mincost;
    solve(s, t, maxflow, mincost);
    if (maxflow != n) printf("NIE\n");
    else printf("%d\n", mincost);
}
```