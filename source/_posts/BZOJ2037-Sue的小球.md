title: "[Solution][BZOJ2037][SDOI2008]Sue的小球"
date: 2015-04-06 15:32:30
tags: [BZOJ,SDOI,DP]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=2037)

## 分析
刚开始想起来有些蛋疼，因为这个问题似乎会有后效性……

不过仔细想一想就会发现，人走过的地方，小球一定都被收集完了。所以已经被收集了小球的区域一定是连续的！并且人每次决策会从区域的一端走向另一端或者就在这一端向外扩展（也就是说最后状态一定是在区域的一端）。离散化以后我们可以以**收集了小球的区域**和当前人在哪一端为状态进行`DP`，`DP`的内容为走到目前为止所有的小球被**浪费**了的价值。最后用总的价值减去浪费了的价值即可。

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

const int B = 0;
const int F = 1;

const int MaxN = 1200;
const int MaxX = 4e4 + 100;
const double eps = 1e-12;
const double inf = 1e50;

double f[MaxN][MaxN][2];
double cnt[MaxX], sum_b[MaxN], sum_f[MaxN];
int n, top_b, top_f, x0;
int x[MaxN], y[MaxN], v[MaxN];
int x_b[MaxN], x_f[MaxN];

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

int main() {
    n = getnum(); x0 = getnum();
    for (int i = 1; i <= n; i++) x[i] = getnum();
    for (int i = 1; i <= n; i++) y[i] = getnum();
    for (int i = 1; i <= n; i++) v[i] = getnum();

    const int MID = 2e4 + 10;
    const int LEN = 2e4;

    double sum = 0;
    for (int i = 1; i <= n; i++) {
        sum += (double)y[i] / 1000;
        cnt[MID + x[i] - x0] += (double)v[i] / 1000;
    }
    cnt[MID] += 1;
    sum += 1;
    for (int i = MID - 1; i >= MID - LEN; i--)
        if (cnt[i] > eps)
            top_b++, sum_b[top_b] = cnt[i], x_b[top_b] = i;
    for (int i = MID; i <= MID + LEN; i++)
        if (cnt[i] > eps)
            top_f++, sum_f[top_f] = cnt[i], x_f[top_f] = i;
    for (int i = top_b - 1; i > 0; i--)
        sum_b[i] += sum_b[i + 1];
    for (int i = top_f - 1; i > 0; i--)
        sum_f[i] += sum_f[i + 1];

    for (int i = 0; i <= top_b; i++)
        for (int j = 0; j <= top_f; j++)
            f[i][j][0] = f[i][j][1] = inf;
    f[0][1][F] = 1;
    
    for (int i = 0; i <= top_b; i++)
        for (int j = 1; j <= top_f; j++) {
            if (!i && j == 1) continue;
            double &curb = f[i][j][B];
            if (i) {
                curb = min(curb, f[i - 1][j][B] + (sum_b[i] + sum_f[j + 1]) * (x_b[i - 1] - x_b[i]));
                curb = min(curb, f[i - 1][j][F] + (sum_b[i] + sum_f[j + 1]) * (x_f[j] - x_b[i]));
            }

            double &curf = f[i][j][F];
            curf = min(curf, f[i][j - 1][F] + (sum_b[i + 1] + sum_f[j]) * (x_f[j] - x_f[j - 1]));
            curf = min(curf, f[i][j - 1][B] + (sum_b[i + 1] + sum_f[j]) * (x_f[j] - x_b[i]));
        }

    double ans = sum - min(f[top_b][top_f][B], f[top_b][top_f][F]);
    printf("%.3f\n", ans);
}
```