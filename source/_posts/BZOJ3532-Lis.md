title: "[Solution][BZOJ3532][SDOI2014]Lis"
date: 2014-12-27 14:11:23
tags: [BZOJ,SDOI,网络流,最小割]
categories: 题解
---
最小割求方案
<!--more-->
## 题目
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3532)

## 分析
最小代价可以用最小割求解。将每一个数字拆点，所有可以作为最长上升子序列起点的数的左节点与原点连边，可以作为终点的数的右节点与汇点连边。如果数$A$在以数$B$结尾的最长链中可以成为数$B$的前缀，那么数$A$的右节点连数$B$的左节点。除了拆开的两个点之间的边容量为代价外，其余的边容量为$inf$（不可以割）。这样割的含义为将所有能构成的最长上升子序列割断。

那么方案怎么办？既然求的是按附加属性排序字典序最小，则先排序，然后从小到大枚举。**满足一条边在割中的充要条件是这条边满流且在残余网络中不存在一条$u$到$v$的路径。**选中一条割边后要将它从图中删掉然后重新网络流。当然这样肯定会`TLE`。正确的处理方法为**找一条$S-u-v-T$的路径，将其退流$1$，然后将$u-v$及反向边的残余容量都设为$0$。**

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
const int MAXN = 720;
const int MAXNODE = 1600;
const int MAXM = 2 * 490000 + 2 * 1600;
struct adata {
    int value, cost, charac, loc;    
    bool operator < (const adata other) const { return  charac < other.charac; }
}a[MAXN];
int f[MAXN], dy[MAXN], n, node;
int point[MAXNODE], nxt[MAXM], v[MAXM], remain[MAXM], tot;
int lastedge[MAXNODE], deep[MAXNODE], num[MAXNODE], cur[MAXNODE];
bool vis[MAXNODE]; queue<int> q;
int ans[MAXN], totans = 0;
inline void memch() {
}
inline void init() {
    memset(point, -1, sizeof(point));
    memset(nxt, -1, sizeof(nxt)); tot = -1;
}
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
inline void addedge(int x, int y, int cap) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y; remain[tot] = cap;
    tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x; remain[tot] = 0;
    //printf("%d %d %d\n", x, y, cap);
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
    memset(vis, 0, sizeof(vis));
    memset(deep, 0, sizeof(deep)); 
    while (!q.empty()) q.pop(); q.push(t); vis[t] = true; deep[t] = 0;
    while (!q.empty()) {
        int now = q.front(); q.pop();
        for (int tmp = point[now]; tmp != -1; tmp = nxt[tmp])
            if (!vis[v[tmp]] && remain[tmp ^ 1]) {
                vis[v[tmp]] = true; deep[v[tmp]] = deep[now] + 1;
                q.push(v[tmp]);
            }
    }
}
inline int isap(int s, int t) {
    int now = s, ans = 0;
    memset(num, 0, sizeof(num)); bfs(s, t);
    for (int i = 1; i <= node; i++) num[deep[i]]++;
    for (int i = 1; i <= node; i++) cur[i] = point[i];
    memset(lastedge, 0, sizeof(lastedge));
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
            num[deep[now] = minn + 1]++; cur[now] = point[now];
            now = v[lastedge[now] ^ 1];
        }
    } return ans;
}
inline bool iscut(int now, int tar) {
    if (now == tar) return true; vis[now] = true;
    for (int tmp = point[now]; tmp != -1; tmp = nxt[tmp])
        if (remain[tmp] && !vis[v[tmp]]) {
            if (iscut(v[tmp], tar)) return true;
        }
    return false;
}
inline bool reflow(int now, int tar, bool rever, int notgo) {
    if (now == tar) return true; vis[now] = true;
    for (int tmp = point[now]; tmp != -1; tmp = nxt[tmp]) {
        if (tmp == notgo || tmp == (notgo ^ 1)) continue;
        if (rever) {
            if (remain[tmp] && !vis[v[tmp]]) {
                if (reflow(v[tmp], tar, rever, notgo)) {
                    remain[tmp]--; remain[tmp ^ 1]++; return true;
                }
            }
        }
        else {
            if (remain[tmp ^ 1] && !vis[v[tmp]])
                if (reflow(v[tmp], tar, rever, notgo)) { 
                    remain[tmp]++; remain[tmp ^ 1]--; return true;
                }
        }
    }
    return false;
}
int main() {
    int testcase = getnum();
    while (testcase--) {
        n = getnum(); init(); node = 2 * n + 2;
        int s = node - 1, t = node;
        for (int i = 1; i <= n; i++) a[i].value = getnum();
        for (int i = 1; i <= n; i++) a[i].cost = getnum();
        for (int i = 1; i <= n; i++) a[i].charac = getnum(), a[i].loc = i;
        f[1] = 1; int maxans = 1;
        for (int i = 2; i <= n; i++) { f[i] = 1;
            for (int j = 1; j < i; j++)
                if (a[j].value < a[i].value) f[i] = max(f[i], f[j] + 1);
            for (int j = 1; j < i; j++)
                if (a[j].value < a[i].value && f[j] + 1 == f[i]) addedge(n + j, i, inf);
            maxans = max(maxans, f[i]);
        }
        for (int i = 1; i <= n; i++) {
            if (f[i] == 1) addedge(s, i, inf);
            else if (f[i] == maxans) addedge(n + i, t, inf);
            dy[i] = tot + 1; addedge(i, i + n, a[i].cost);
        }
        printf("%d ", isap(s, t));
        sort(a + 1, a + 1 + n); totans = 0;
        for (int i = 1; i <= n; i++) { int now = a[i].loc;
            if (remain[dy[now]]) continue; memset(vis, 0, sizeof(vis));
            if (!iscut(now, now + n)) {
                ans[++totans] = now; memset(vis, 0, sizeof(vis));
                reflow(now, s, true, dy[now]); memset(vis, 0, sizeof(vis));
                reflow(n + now, t, false, dy[now]); remain[dy[now]] = remain[dy[now] ^ 1] = 0;
            }
        }
        printf("%d\n", totans); sort(ans + 1, ans + 1 + totans);
        for (int i = 1; i < totans; i++) printf("%d ", ans[i]);
        printf("%d\n", ans[totans]);
    }
}
```