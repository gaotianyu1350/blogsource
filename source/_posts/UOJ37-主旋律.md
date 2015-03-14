title: "[Solution][UOJ37][THU Training 2014]主旋律"
date: 2015-02-22 23:14:05
tags: [UOJ,计数问题,容斥原理]
---
<!--more-->
## 题目描述
[传送门](http://uoj.ac/problem/37)

## 分析
题目描述还真是主旋律啊……

题目给定一张简单有向图，问有多少边子集满足构成一个强联通图。求强连通显然太复杂了，所以反向考虑问题，求有多少边子集不是强连通。不是强连通的可以将强连通分量缩点，缩点后整张图变成了一个`DAG`。求`DAG`方案数，可以枚举有哪些点（哪些强连通分量）是出度为$0$的点，答案为
$$ D[S]=\sum_{T\subseteq S~|T|\geq 1} (-1)^{|T|-1}ways[T,S-T] D[S-T] $$ 
其中$D[S]$表示集合$S$构成`DAG`的方案数，$ways[B,A]$表示从$A$到$B$连边的方案数。*（公式来自陈老师pdf~）*

这道题的核心思想就是这样，但是还有很多细节需要考虑。首先必须要将算法的复杂度优化到$O(3^n)$，也就是只能有`集合DP`的复杂度，不能有别的多余的枚举。那我们就必须要把强连通分量的枚举和$ways$数组的计算放到`DP`里面去搞。令$f[T]$表示以$T$集合的点作为汇点的方案数。注意这个统计是带方向的。就是说，当强连通分量的个数是奇数时，放进去的是正数，反之是偶数，**因为容斥原理里面系数的正负不是根据集合里点的个数，而是具体的方案里面强连通分量的个数（也就是缩点后点的个数）定的，如果不在这里提前统计会很麻烦**。$ways$的计算需要一些技巧，我们可以用$int$来存储每个点对应的出边和入边，然后根据上一个状态推下一个状态的$ways$，细节见程序。

总之这是一道思路里面处处巧妙的题，不愧是陈老师出的题啊……

## 代码
```c++
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <cmath>
#include <iostream>
#include <algorithm>
using namespace std;
typedef long long ll;
const int MaxN = 20;
const int MaxSt = (1 << 15) + 10;
const int Mod = 1e9 + 7;
int n, m;
int pw2[MaxN * MaxN], bitCnt[MaxSt], last[MaxSt];
int edgeFrom[MaxN], edgeTo[MaxN];
int dp[MaxSt], res[MaxSt], f[MaxSt], tmp[MaxSt];
inline void init() {
    pw2[0] = 1;
    for (int i = 1; i < MaxN * MaxN; i++)
        pw2[i] = pw2[i - 1] * 2 % Mod;
    for (int i = 1; i < MaxSt; i++)
        bitCnt[i] = bitCnt[i - (i & -i)] + 1;
    for (int i = 1; i < MaxSt; i++)
        for (int j = 0; j < 15; j++)
            if (i >> j & 1) last[i] = j;
}
inline void add(int &a, int b) {
    a += b;
    if (a >= Mod) a -= Mod;
}
int main() {
    init();
    cin >> n >> m;
    for (int i = 1; i <= m; i++) {
        int x, y;
        cin >> x >> y;
        x--; y--;
        edgeFrom[x] |= (1 << y);
        edgeTo[y] |= (1 << x);
    }
    f[0] = dp[0] = res[0] = 1;
    for (int s = 1; s < (1 << n); s++) {
        int edgeCnt = 0;
        for (int i = 0; i < n; i++)
            if (s >> i & 1) edgeCnt += bitCnt[edgeFrom[i] & s];
        res[s] = pw2[edgeCnt];
        int lastone = last[s];
        for (int pre = s ^ (1 << lastone); pre; pre = (pre - 1) & s) 
            add(f[s], (ll)res[s - pre] * (Mod - f[pre]) % Mod);
        for (int ts = s; ts; ts = (ts - 1) & s) {
            if (ts == s) {
                tmp[ts] = 0;
            } else {
                int lastP = last[s - ts];
                int lastTs = ts | (1 << lastP);
                tmp[ts] = tmp[lastTs] - bitCnt[edgeTo[lastP] & (s - lastTs)]
                                      + bitCnt[edgeFrom[lastP] & ts];
            }
            add(dp[s], (ll)pw2[tmp[ts]] * f[ts] % Mod * dp[s - ts] % Mod);
        }
        add(res[s], Mod - dp[s]);
        add(f[s], res[s]);
        add(dp[s], res[s]);
    }
    cout << res[(1 << n) - 1] << endl;
}
```