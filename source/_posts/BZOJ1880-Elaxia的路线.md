title: "[Solution][BZOJ1880][SDOI2009]Elaxia的路线"
date: 2015-04-03 11:21:04
tags: [BZOJ,SDOI,最短路径,SPFA,DP]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1880)

## 分析
首先跑最短路。然后我们可以按照距离$x_1$的距离进行排序再`DP`。令$f[i][0]$表示$i$为公共路径的一部分，公共路径的最长距离，$f[i][1]$表示到$i$之前最长的公共路径。则

当$e$为$(u,i)$之间的边，且在两条最短路径上
$$ f[i][0]=max(f[u][0]+w(e)) $$

当$(u,i)$在第一条最短路径上
$$ f[i][1]=max(f[u][1]) $$

当然不要忘记$f[i][1]=max(f[i][1],f[i][0])$

要注意这道题目里面公共路径上两个人的方向可以相反！

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

const int MaxN = 1600;
const int MaxM = 2.25e6 * 2 + 100;
const int INF = 1e9;

int point[MaxN], nxt[MaxM], v[MaxM], w[MaxM], tot;
int disA[2][MaxN], disB[2][MaxN], f[MaxN][2], pos[MaxN];
bool check[MaxN];

int n, m, s1, t1, s2, t2;

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

inline void addedge(int x, int y, int ww) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y; w[tot] = ww;
    tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x; w[tot] = ww;
}

inline void spfa(int s, int *dis) {
    for (int i = 1; i <= n; i++)
        dis[i] = INF;
    memset(check, 0, sizeof(check));
    queue<int> q;
    q.push(s); dis[s] = 0; check[s] = true;

    while (!q.empty()) {
        int now = q.front(); q.pop(); check[now] = false;
        for (int tmp = point[now], u; u = v[tmp], tmp; tmp = nxt[tmp])
            if (dis[u] > dis[now] + w[tmp]) {
                dis[u] = dis[now] + w[tmp];
                if (!check[u])
                    check[u] = true, q.push(u);
            }
    }
}

bool cmp(const int &a, const int &b) {
    return disA[0][a] < disA[0][b];
}

inline int solve() {
    for (int i = 1; i <= n; i++)
        pos[i] = i;
    sort(pos + 1, pos + 1 + n, cmp);

    int rdisA = disA[0][t1];
    int rdisB = disB[0][t2];

    for (int i = 1; i <= n; i++) {
        int now = pos[i];
        if (disA[0][now] + disA[1][now] != rdisA) continue;
        for (int tmp = point[now], u; u = v[tmp], tmp; tmp = nxt[tmp]) {
            if (disA[0][u] > disA[0][now]) continue;
            if (disA[0][u] + disA[1][u] != rdisA) continue;

            if (disA[0][u] + w[tmp] == disA[0][now]) {
                if (disB[0][now] + disB[1][now] == rdisB && 
                    disB[0][u] + disB[1][u] == rdisB && 
                    (disB[0][u] + w[tmp] == disB[0][now] || disB[0][now] + w[tmp] == disB[0][u]))
                    f[now][0] = max(f[now][0], f[u][0] + w[tmp]);
            
                f[now][1] = max(f[now][1], f[u][1]);
            }
        }
        f[now][1] = max(f[now][1], f[now][0]);
    }

    return f[t1][1];
}

int main() {
    n = getnum(); m = getnum();
    s1 = getnum(); t1 = getnum();
    s2 = getnum(); t2 = getnum();
    for (int i = 1; i <= m; i++) {
        int u = getnum(); int v = getnum();
        int w = getnum();
        addedge(u, v, w);
    }

    spfa(s1, disA[0]);
    spfa(s2, disB[0]);
    spfa(t1, disA[1]);
    spfa(t2, disB[1]);

    cout << solve() << endl;
}
```