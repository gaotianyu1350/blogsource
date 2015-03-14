title: "[Solution][BZOJ1532][POI2005]Kos-Dicing"
date: 2014-12-27 17:32:35
tags: [BZOJ,POI,网络流,二分]
categories: 题解
---
这数据范围竟然也能跑
<!--more-->
## 题目
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1532)

## 分析
从一个网络流合集里面看到了这道题
看到数据范围就吓傻了……

二分答案+网络流验证。比赛向选手连边，容量$1$。源点向比赛连边，容量$1$，选手向汇点连边，容量为二分的值。

那么问题来了：不会T吗？

优越的`dinic`在二分图中的效率为$O(\sqrt n m)$。一直写`isap`的我只好转战`dinic`了……

**补充：我的`dinic`刚开始写错了……当前弧优化加错了位置，导致`dinic`效率非常低下……**

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

const int inf = 1e9;
const int MAXN = 1e4 + 10;
const int MAXNODE = 2 * 1e4 + 10;
const int MAXM = 2 * 1e4 * 2 + 2 * 1e4 * 2;

int point[MAXNODE], nxt[MAXM], v[MAXM], remain[MAXM], stremain[MAXM], tot;
int cur[MAXNODE], deep[MAXNODE], dy[MAXN], n, m, node;
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
inline bool bfs(int s, int t) {
    memset(deep, 0x7f, sizeof(deep)); deep[s] = 0;
    for (int i = 1; i <= node; i++) cur[i] = point[i];
    while (!q.empty()) q.pop(); q.push(s);
    while (!q.empty()) {
        int now = q.front(); q.pop();
        for (int tmp = point[now]; tmp != -1; tmp = nxt[tmp])
            if (deep[v[tmp]] > inf && remain[tmp])
                deep[v[tmp]] = deep[now] + 1, q.push(v[tmp]);
    } return deep[t] < inf;
}
inline void addedge(int x, int y, int cap) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y; stremain[tot] = cap;
    tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x; stremain[tot] = 0;
}
inline int dfs(int now, int t, int limit) {
    if (!limit || now == t) return limit;
    int flow = 0, f;
    for (int tmp = cur[now]; tmp != -1; tmp = nxt[tmp]) { cur[now] = tmp;
        if (deep[v[tmp]] == deep[now] + 1 && (f = dfs(v[tmp], t, min(limit, remain[tmp])))) {
            flow += f; limit -= f; cur[now] = tmp;
            remain[tmp] -= f; remain[tmp ^ 1] += f;
            if (!limit) break;
        } 
    } return flow;
}
inline int dinic(int s, int t, int value) {
    int ans = 0; for (int i = 0; i <= tot; i++) remain[i] = stremain[i];
    for (int i = 1; i <= n; i++) remain[dy[i]] = value;
    while (bfs(s, t)) ans += dfs(s, t, inf); return ans;
}
int main() {
    init();
    n = getnum(); m = getnum();
    node = n + m + 2; int s = node - 1, t = node;
    for (int i = 1; i <= m; i++) {
        int x, y; x = getnum(); y = getnum();
        addedge(s, i, 1); addedge(i, m + x, 1); addedge(i, m + y, 1);
    }
    for (int i = 1; i <= n; i++) dy[i] = tot + 1, addedge(m + i, t, 0);
    int l = 0, r = m;
    while (l < r) {
        int mid = (l + r) >> 1;
        if (dinic(s, t, mid) == m) r = mid; else l = mid + 1;
    } printf("%d\n", l);
}
```