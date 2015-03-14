title: "[Solution][BZOJ1391][CEOI2008]order"
date: 2014-12-24 17:07:13
tags: [BZOJ,CEOI,最小割,网络流,最大权闭合子图]
categories: 题解
---
好久没做网络流了，今天刷道题练练手
<!--more-->
## 题目
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1391)

## 分析
典型的最大权闭合子图用最小割解决的模型。将所有的机器与$S$连边，容量为机器购买的费用，将所有的任务与$T$连边，容量为该任务的收入。再将任务与它需要的机器连边，容量为租用该机器的费用。然后对这张图求最小割，用所有任务的收入减去最小割即为所求答案。

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <queue>
#include <algorithm>
using namespace std;
const int MAXN = 2 * 1e3;
const int MAXNODE = 3 * 1e3;
const int MAXM = 3 * 1e6;
const int inf = 1e9;
int point[MAXNODE], nxt[MAXM], v[MAXM], remain[MAXM], tot;
int deep[MAXNODE], cur[MAXNODE], lastedge[MAXNODE], num[MAXNODE];
bool vis[MAXNODE];
int n, m, node;
queue<int> q;
inline void memch() {
}
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
inline void addedge(int x, int y, int cap) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y; remain[tot] = cap;
    tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x; remain[tot] = 0;
}
inline int addflow(int s, int t) {
    int now = t, ans = inf;
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
inline void bfs(int s, int t) {
    memset(deep, 0, sizeof(deep)); memset(vis, 0, sizeof(vis));
    while (!q.empty()) q.pop(); q.push(t); vis[t] = true;
    deep[t] = 0;
    while (!q.empty()) {
        int now = q.front(); q.pop();
        for (int tmp = point[now]; tmp != -1; tmp = nxt[tmp])
            if (!vis[v[tmp]] && remain[tmp ^ 1]) {
                deep[v[tmp]] = deep[now] + 1;
                vis[v[tmp]] = true; q.push(v[tmp]);
            }
    }
}
inline int isap(int s, int t) {
    int ans = 0, now = s;
    bfs(s, t); memset(num, 0, sizeof(num));
    memset(lastedge, 0, sizeof(lastedge)); memset(cur, 0, sizeof(cur));
    for (int i = 1; i <= node; i++) num[deep[i]]++;
    for (int i = 1; i <= node; i++) cur[i] = point[i];
    while (deep[s] < node) {
        if (now == t) {
            ans += addflow(s, t);
            now = s;
        } bool isok = false;
        for (int tmp = cur[now]; tmp != -1; tmp = nxt[tmp])
            if (deep[now] == deep[v[tmp]] + 1 && remain[tmp]) {
                isok = true; lastedge[v[tmp]] = tmp; cur[now] = tmp;
                now = v[tmp]; break;
            }
        if (!isok) { int minn = node - 1;
            for (int tmp = point[now]; tmp != -1; tmp = nxt[tmp])
                if (remain[tmp]) minn = min(minn, deep[v[tmp]]);
            if (!(--num[deep[now]])) break;
            num[deep[now] = minn + 1]++;
            cur[now] = point[now]; now = v[lastedge[now] ^ 1];
        }
    } return ans;
}
int main() {
    init();
    n = getnum(); m = getnum(); node = n + m + 2;
    int s = node - 1, t = node, summoney = 0;
    for (int i = 1; i <= n; i++) {
        int x, y; summoney += x = getnum(); y = getnum(); 
        addedge(m + i, t, x);
        for (int j = 1; j <= y; j++) {
            int no, rent; no = getnum(); rent = getnum();
            addedge(no, m + i, rent);           
        }
    } 
    for (int i = 1; i <= m; i++) {
        int x = getnum(); addedge(s, i, x);
    } printf("%d\n", summoney - isap(s, t));
}
```