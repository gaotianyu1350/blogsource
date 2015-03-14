title: "[Solution][BZOJ1073][SCOI2007]KShort"
date: 2014-12-04 19:22:33
tags: [k短路,DFS,BZOJ,SCOI]
categories: 题解
---
不cheat不cheat
<!--more-->
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1073)

## 分析
这道题有好几种做法。有`二分`然后判断是否合法的（似乎要cheat才能过），有用`A*`的。我用的是网上一位神犇的做法。先预处理出所有点$i$到终点$t$的最短距离`dis[i]`。设定一个限定值`dis_limit`，要求此次`dfs`的时候距离不能超过它。再定义一个`min_outlimit_dis`表示`dfs`经过的点里面到$t$的期望最短距离超过`dis_limit`的最小值。详细解释一下：假设此时`dfs`到了点`now`，已经走过了距离`now_dis`，如果`now_dis + dis[now] > dis_limit`，说明这种状态下这个点再怎么走也不可能到达终点，而`now_dis + dis[now]`是当前状态下可能的最小距离，我们用它更新`min_outlimit_dis`，同时可行性剪枝。如果一次`dfs`下来，能够走到$t$的路径的条数不足$k$，那么就调高`dis_limit`为`min_outlimit_dis`，这样保证调整的范围最小，同时又不至于太小导致超时。如果一次`dfs`的时候路径条数大于等于$k$，那么输出方案。如何输出呢？首先`dfs`的时候要先走序号小的点，这样保证了字典序。每次记录一下上一次`dfs`得到的路径条数。这一次获得的路径条数可能不全（因为一到$k$就停止了），但是上一次获得的一定是全的（因为不到$k$），那么多获得的这些一定是和目标路径长度一样的（这一次刚刚更新出来的）。这一次的个数减去上一次的个数，就是在这些长度等于`dis_limit`的最新更新出来的路径里面，答案是第几条（因为是按照字典序更新的，所以答案前面的和它长度一样长的路径一定都更新出来了，这个地方比较绕，可以自己多想一想）。
*细节1* 即使是`now_dis`已经超过了`dis_limit`，也要尝试让`now_dis + dis[now]`去更新一下`min_outlimit_dis`。否则有可能会导致`min_outlimit_dis`无法得到更新的情况。
*细节2* 每次往下`dfs`之前，要先判断一下从这个点在当前状态下（也就是要考虑已经走过的那些点不能走）是否可以到达$t$。这个可以避免第$4$个点超时，但是会增大其它点的复杂度
总之这是一个不`cheat`的做法。。。

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
#include <vector>
#include <queue>
using namespace std;
typedef long long ll;
const int MAXN = 60 * 2;
const int MAXM = 60 * 60 * 2;
const int MAXK = 210 * 2;
const int inf  = 1e9;
struct ansdata {
    int answay[MAXN], ansdis;
    bool operator < (const ansdata other) {
        if (ansdis != other.ansdis) return ansdis < other.ansdis;
        int tar = min(answay[0], other.answay[0]);
        for (int i = 1; i <= tar; i++)
            if (answay[i] != other.answay[i]) return answay[i] < other.answay[i];
        return false;
    }
}ans[MAXK];
int point[MAXN] = {0}, nxt[MAXM] = {0}, v[MAXM] = {0}, w[MAXM] = {0}, tot = 0;
int rpoint[MAXN] = {0}, rnxt[MAXM] = {0}, rv[MAXM] = {0}, rw[MAXM] = {0}, rtot = 0;
int dis[MAXN], way[MAXN], check[MAXN] = {0};
bool inq[MAXN];
int n, m, k, s, t, curlimit, minout, cnt, tt, dfst;
struct edge {
    int x, y, w;
    bool operator < (const edge other) const {
        return x < other.x || (x == other.x && y > other.y);
    }
}e[MAXM];
inline void addedge(int x, int y, int ww) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y; w[tot] = ww;
}
inline void raddedge(int x, int y, int ww) {
    rtot++; rnxt[rtot] = rpoint[x]; rpoint[x] = rtot; rv[rtot] = y; rw[rtot] = ww;
}
inline void spfa() {
    queue<int> q;
    while (!q.empty()) q.pop();
    memset(inq, 0, sizeof(inq));
    memset(dis, 0x7f, sizeof(dis));
    dis[t] = 0; inq[t] = true; q.push(t);
    while (!q.empty()) {
        int now = q.front(); q.pop(); inq[now] = false;
        for (int tmp = rpoint[now]; tmp; tmp = rnxt[tmp])
            if (dis[rv[tmp]] > dis[now] + rw[tmp]) {
                dis[rv[tmp]] = dis[now] + rw[tmp];
                if (!inq[rv[tmp]])
                    inq[rv[tmp]] = true, q.push(rv[tmp]);
            }
    }
}
bool dfs(int now, int curdis, int loc) {
    way[tt = loc] = now;
    if (curdis + dis[now] > curlimit) {
        minout = min(minout, curdis + dis[now]);
        return false;
    }
    if (now == t) {
        cnt++;
        memcpy(ans[cnt].answay + 1, way + 1, sizeof(int) * loc);
        ans[cnt].answay[0] = loc;
        ans[cnt].ansdis = curdis;
        if (cnt >= k) return true;
    }
    queue<int> q; q.push(now); check[now] = ++dfst;
    while (!q.empty()) {
        int now = q.front(); q.pop();
        for (int tmp = point[now]; tmp; tmp = nxt[tmp])
            if (!inq[v[tmp]] && check[v[tmp]] != dfst)
                check[v[tmp]] = dfst, q.push(v[tmp]);
    }
    if (check[t] != dfst) return false;
    for (int tmp = point[now]; tmp; tmp = nxt[tmp])
        if (!inq[v[tmp]]) {
            inq[v[tmp]] = true;
            if (dfs(v[tmp], curdis + w[tmp], loc + 1)) return true;
            inq[v[tmp]] = false;
        }
    return false;
}
inline void solve() {
    int lastcnt = 0;
    curlimit = dis[s];
    while (1) {
        minout = inf; cnt = 0;
        memset(inq, 0, sizeof(inq));
        memset(check, 0, sizeof(check));
        inq[s] = true; dfst = 0;
        if (dfs(s, 0, 1)) {
            lastcnt = cnt - lastcnt;
            for (int i = 1; i <= cnt; i++)
                if (ans[i].ansdis == curlimit) {
                    lastcnt--;
                    if (!lastcnt) {
                        for (int j = 1; j <= ans[i].answay[0]; j++) {
                            printf("%d", ans[i].answay[j]);
                            if (j < ans[i].answay[0]) putchar('-');
                        }
                        putchar('\n');
                        break; 
                    }
                }
            break;
        } else if (minout == inf) {
            printf("No\n");
            break;
        } else
            curlimit = minout, lastcnt = cnt;
    }
}
int main() {
    scanf("%d%d%d%d%d", &n, &m, &k, &s, &t);
    for (int i = 1; i <= m; i++) scanf("%d%d%d", &e[i].x, &e[i].y, &e[i].w);
    sort(e + 1, e + 1 + m);
    for (int i = 1; i <= m; i++) {
        addedge(e[i].x, e[i].y, e[i].w);
        raddedge(e[i].y, e[i].x, e[i].w);
    }
    spfa();
    solve();
}
```