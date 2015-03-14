title: "[Solution][BZOJ1131][POI2008]Sta"
date: 2015-02-04 16:41:35
tags: [BZOJ,POI]
categories: 题解
---
<!--more-->
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1131)

## 分析
用$sumdown(i)$表示以$1$为根的树中，以$i$为根的子树的深度之和，$size(i)$表示以$1$为根的树种，以$i$为根的子树的节点个数，$sum(i)$则表示以$i$为根的树的深度和。显然$sum(1)=sumdown(1)$。对于$i\neq 1$，有
$$ sum(i)=sum(father(i))-size(i)+n-size(i) $$
式子很好理解，就不解释了

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
typedef long long ll;
const int MaxN = 1e6 + 10;
const ll inf = 1e18;
int point[MaxN], v[MaxN * 2], nxt[MaxN * 2], tot = 0, n;
int size[MaxN], father[MaxN], cur[MaxN];
ll sumdown[MaxN], sum[MaxN];
queue<int> q;
inline void addedge(int x, int y) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y;
    tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x;
}
inline void dfs() {
    int now = 1;
    memset(cur, -1, sizeof(cur));
    while (now) {
        if (cur[now] == -1) {
            cur[now] = point[now];
            size[now] = 1;
        }
        else {
            size[now] += size[v[cur[now]]];
            sumdown[now] += sumdown[v[cur[now]]] + size[v[cur[now]]]; 
            cur[now] = nxt[cur[now]];
        }
        if (v[cur[now]] == father[now])
            cur[now] = nxt[cur[now]];
        if (!cur[now])
            now = father[now];
        else {
            int u = v[cur[now]];
            father[u] = now;
            now = u;
        }
    }
}
inline int bfs() {
    q.push(1); sum[1] = sumdown[1];
    ll maxsum = 0; int wh = 0;
    while (!q.empty()) {
        int now = q.front(); q.pop();
        if (sum[now] > maxsum || (sum[now] == maxsum && now < wh))
            maxsum = sum[now], wh = now;
        for (int tmp = point[now]; tmp; tmp = nxt[tmp])
            if (v[tmp] != father[now]) {
                int u = v[tmp];
                sum[u] = sum[now] - size[u] + n - size[u];
                q.push(u);
            }
    }
    return wh;
}
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
int main() {
    n = getnum();
    for (int i = 1; i < n; i++) {
        int x, y;
        x = getnum(); y = getnum();
        addedge(x, y);
    }
    dfs();
    cout << bfs() << endl;
}
```