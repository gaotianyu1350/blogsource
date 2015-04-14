title: "[Solution][BZOJ1875][SDOI2009]HH去散步"
date: 2015-04-01 14:24:03
tags: [BZOJ,SDOI,DP,矩阵乘法]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1875)

## 分析
表示不明白为什么不能直接用点DP……（交上去莫名RE囧）

这道题最蛋疼的地方就是不能走刚刚走过的那条边。如果把原来的边当成“点”，原来的点当成连接“点”的“边”，显然就无需考虑这个问题（不可能这个“点”又转移到这个“点”）。然后再用矩阵乘法优化即可。

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

typedef long long ll;
const int MaxN = 150;
const int MaxT = 240;
const int Mod = 45989;

int n, m, t, start, end;

inline void update(int &x, int y) {
    x = (x + y % Mod) % Mod;
}

struct matrix {
    int a[MaxN][MaxN];
    int n, m;

    matrix(int _n, int _m) {
        n = _n, m = _m;
        memset(a, 0, sizeof(a));
    }

    matrix operator = (const matrix &o) {
        n = o.n, m = o.m;
        memcpy(a, o.a, sizeof(o.a));
        return *this;
    }

    matrix operator * (const matrix &o) const {
        matrix ans(n, o.m);
        for (int i = 1; i <= ans.n; i++)
            for (int j = 1; j <= ans.m; j++)
                for (int k = 1; k <= m; k++)
                    update(ans.a[i][j], (ll)a[i][k] * o.a[k][j] % Mod);
        return ans;
    }

    matrix operator ^ (int p) const {
        matrix ans(n, m);
        matrix tmp(n, m);
        for (int i = 1; i <= n; i++) ans.a[i][i] = 1;
        tmp = *this;

        for (; p; p >>= 1, tmp = tmp * tmp)
            if (p & 1)
                ans = ans * tmp;
        return ans;
    }
};

int point[MaxN], nxt[MaxN], v[MaxN], tot;

inline void init() {
    tot = -1;
    memset(point, -1, sizeof(point));
}

inline void addedge(int x, int y) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y;
    tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x;
}

int main() {
    init();

    n = getnum(); m = getnum(); t = getnum(); 
    start = getnum() + 1; end = getnum() + 1;

    for (int i = 1; i <= m; i++) {
        int x = getnum() + 1, y = getnum() + 1;
        addedge(x, y);
    }

    matrix trans(m * 2, m * 2);
    for (int i = 1; i <= n; i++) {
        for (int tmp1 = point[i]; tmp1 != -1; tmp1 = nxt[tmp1])
            for (int tmp2 = point[i]; tmp2 != -1; tmp2 = nxt[tmp2])
                if (tmp1 != tmp2) {
                    int idX = tmp1 ^ 1, idY = tmp2;
                    trans.a[idY + 1][idX + 1] = 1;
                }

    }

    matrix ans(m * 2, 1);
    for (int tmp = point[start]; tmp != -1; tmp = nxt[tmp])
        ans.a[tmp + 1][1] = 1;
    
    trans = trans ^ (t - 1);
    ans = trans * ans;

    int finalAns = 0;
    for (int tmp = point[end]; tmp != -1; tmp = nxt[tmp])
        update(finalAns, ans.a[(tmp ^ 1) + 1][1]);
    printf("%d\n", finalAns);
}
```