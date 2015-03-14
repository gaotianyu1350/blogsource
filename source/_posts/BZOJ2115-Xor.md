title: "[Solution][BZOJ2115][WC2011]Xor"
date: 2014-12-20 07:45:22
tags: [BZOJ,WC,xor方程组]
categories: 题解
---
无向连通图xor和最大路径
<!--more-->
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=2115)

## 分析
详细分析见莫涛的[高斯消元解异或方程组](http://wenku.baidu.com/link?url=GEdOCrRk1KcIOvLdiVES8GhfbVOjnnIJkYbpLQyiSpm9BtxKjfLyV4-NXXPi8DRE3FS4jejTHDqy3n8uXTqy-UNSBxsCnWJn78gS10Zzl2e)。
大体思路是先求出图中所有的独立环（就是不能由别的环凑出来的环）的xor和。注意这里的环是可以走重复路径的。任意两条从$1$到$n$的路径，都能构成一个环。相反，任意一条路径都可以由一条固定的路径和一些环组成。如果是环和路径不相交，可以想象成从这个路径的某一点走到环上，然后再原路返回。这样问题相当于变成了从一组数中选出若干个使得xor和最大，用线性基即可。
如何求出独立环？根据莫涛ppt里面的推理，先从图中提取出一棵生成树，每往上加一条边，只会多一个独立环。所以bfs或者dfs的时候如果遇到已经访问过的点把这个环加进去即可。注意用这种方法会得到$2m-n+1$个环，数组一定要开够！

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
const ll MAXN = 5 * 1e4 + 10;
const ll MAXM = 1e5 + 10;
const ll MAXBIT = 62;
ll n, m;
ll point[MAXN] = {0}, nxt[MAXM * 2] = {0}, v[MAXM * 2] = {0}, tot = 0;
ll w[MAXM * 2] = {0};
ll a[MAXM * 2] = {0}, lb[MAXM] = {0}, dis[MAXN] = {0}, atot = 0;
bool vis[MAXN] = {0}; ll f[MAXN] = {0};
queue<int> q;
inline void calcLB() {
    for (ll i = 1; i <= atot; i++)
        for (ll j = MAXBIT; j >= 0; j--)
            if (a[i] & (1LL << j)) {
                if (!lb[j]) { lb[j] = a[i]; break; }
                else a[i] ^= lb[j];
            }
} 
inline ll getnum() {
    ll ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
} 
inline void addedge(ll x, ll y, ll ww) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y; w[tot] = ww;
    tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x; w[tot] = ww;
}
int main() {
    n = getnum(); m = getnum();
    ll x, y; ll ww;
    for (ll i = 1; i <= m; i++) {
        x = getnum(); y = getnum(); ww = getnum();
        addedge(x, y, ww);
    }
    while (!q.empty()) q.pop(); q.push(1); vis[1] = true;
    while (!q.empty()) {
        ll now = q.front(); q.pop();
        for (ll tmp = point[now]; tmp; tmp = nxt[tmp])
            if (!vis[v[tmp]]) {
                dis[v[tmp]] = dis[now] ^ w[tmp]; f[v[tmp]] = now; vis[v[tmp]] = true;
                q.push(v[tmp]);
            }else if (v[tmp] != f[now]) a[++atot] = (w[tmp] ^ dis[now] ^ dis[v[tmp]]);
    }
    calcLB();
    ll ans = dis[n];
    for (ll i = MAXBIT; i >= 0; i--)
        if ((ans ^ lb[i]) > ans) ans ^= lb[i];
    printf("%lld\n", ans);   
}
```