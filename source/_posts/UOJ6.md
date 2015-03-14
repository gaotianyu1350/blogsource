title: "[Contest][UOJ#6]2015-3-8"
date: 2015-03-09 20:35:02
tags: [Hash,UOJ,构造,逻辑推理,生成树计数,随机]
categories: 比赛
---
## 题目描述
[传送门](http://uoj.ac/contest/9)

## A
假设第$i$位上的字母为$a$。可以得到方程
$$ 26h\_i-26^na+a=h\_{i+1} $$
化简得
$$ (1-26^n)a=h\_{i+1}-26h\_i $$

这样我们就可以在$O(n)$的时间内求出整个序列。等等！如果$26^n=1$，$1-26^n$就没有逆元了！这该怎么办……

如果$26^n=1$，就会导致$h\_{i+1}=26h\_i$，这样的话不论放什么字母都可以。我们只需要把$h\_0$作为字符串`hash`还原一个字符串就可以啦^-^

比赛的时候完全没有考虑解方程，直接枚举验证，所以忽略了没有逆元的情况……

## B
这道题的部分分很科学啊~

对于1、2，直接一个环就好了。（特判$0$的情况，一个不连通的图即可）。

对于3、4，暴力

对于5、6、7，有两个点好像是完全图（MD这我怎么猜得到……），剩下一个分解质因数发现和事小于$100$的，然后就构造几个环串起来就好了。

标算？好吧太丧心病狂了……随机$700$个（原题解是$1000$个，我会`TLE`……）$n=12$的图，然后对于每一次询问，找出四个图，使得它们的方案数乘积为要求的数。可以先把两两图之间的乘积压到`hash`里面，然后再枚举两个图，用逆元求出剩下两个图，再去`hash`里面找。原题解分析了一大堆，证明有很大概率这样能够覆盖模数下的所有整数答案……（前提是你要选好随机种子，还要在`TLE`和`WA`之间找到一个良好平衡~）

## C
我的好多非OI同学似乎都听说过这道题的原版……所以当我问他们，他们秒出正解的时候简直吓尿了……不过UOJ上的这道题还是蛮丧心病狂的，理解起来很不智商友好啊……

核心思路是这样的：一个有病狗的人判断自己什么时候应该开枪，他会先假设自己的狗没有病，然后看在这种情况下剩下的人应该什么时候开枪。如果剩下的人没有在那个时候开枪（一只病狗都没有的情况开枪时间为$0$），就说明自己的狗是病狗，于是开枪杀死。而一个状态的开枪时间，是所有的人中开枪时间最早的那一个。

于是我们有了一个`状压DP`的思路。首先约定黑点为病狗，白点为健康的狗。$dp\_S$表示状态$S$下的开枪时间。枚举每一个有病狗的人，然后找前面那个状态。那些他能看到的人状态保持不变，自己变白。那些看不见的点呢？只好暴力枚举了。把这些状态里面最晚的时间$+1$作为这个人的开枪时间，再把这些人里面最早的作为这个状态的开枪时间。注意：如果转移的时候出现了环，就说明环上的所有状态都不会开枪~

然后原题解说打了一个表，发现对于任意状态$S\subset V$，都有$dp\_S<dp\_V$。这样的话我们对于每个人，找下一个状态就可以直接把所有他看不到的点变成黑点了。

然后我们会发现这似乎是图上一些黑点，然后黑点变成白点，后继变成黑点bulabula……下面我们来详细研究一下模型的转换。

首先要先把环去掉，而且黑点是不可能存在在环上的（否则没有人开枪）。这样我们就得到了一个`DAG图`。（注意这个地方删除环的时候要有技巧，要从出度为$0$的点开始倒着`DP`。因为如果环在`DAG`的汇点位置方案同样是不可行的。）现在模型可以变为，`DAG`上有一些黑点，可以选择一个黑点变成白点， 然后把它所有的儿子变成黑点，最后所有点都会变成白点（对应`DP`中没有病狗，时间为$0$的情况）。而曾经有$k$个点被染黑，时间就是$k$。

好吧说实话这个非常非常不好理解（我想了快一天了……）。所有人里面取时间最短的体现在每个点变黑只算一次，枚举狗主人体现在选择黑点然后变白……

如果这样的话其实问题就是求对于`DAG`上的一个点集，它们可以直接或者间接的走到多少点上。对于一个点，如果它能够被$i$个点走到，那么它对时间答案的贡献为$(2^i-1)2^{n-i}$（注意$n$不包括环上的点）。它对死狗的贡献为$2^{n-i}$（因为这样它必须不被除自己以外的点走到）。可以在$O(n^3)$的时间内求出来。如果加上`bitset`优化，就可以过了。（一定记得加读入挂啊~）

说了半天感觉还是没有说清楚，还是看出题人的题解叭~

[原题解传送门](http://vfleaking.blog.uoj.ac/blog/180)

<!--more-->
## 代码
A
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
using namespace std;
 
typedef long long ll;
const int MaxN = 1e5 + 10;
 
int cm[MaxN], n, m, h[MaxN];
char s[MaxN];
 
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
 
inline void init(int N) {
    cm[0] = 1;
    for (int i = 1; i <= N; i++)
        cm[i] = (ll)cm[i - 1] * 26 % m;
}
 
int main() {
    n = getnum(); m = getnum();
    init(n);
    for (int i = 0; i < n; i++) h[i] = getnum() % m;
    h[n] = h[0];
 
    if (cm[n] == 1) {
        for (int i = 0; i < n; i++) {
            s[n - i - 1] = 'a' + h[0] % 26;
            h[0] /= 26;
        }
        printf("%s\n", s);
        return 0;
    }
 
    for (int i = 0; i < n; i++)
        for (int ans = 0; ans < 26; ans++) {
            int tmp = (ll)ans * cm[n - 1] % m;
            int nxt = (ll)(h[i] + m - tmp) % m * 26 % m;
            nxt = (nxt + ans) % m;
            if (nxt == h[i + 1]) {
                putchar(ans + 'a');
                break;
            }
        }
    putchar('\n');
}
```

B
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
using namespace std;
typedef long long ll;
 
int mod = 998244353;
 
inline double getRand() {
    return (double)rand() / RAND_MAX;
}
 
inline ll qE(ll a, int p) {
    ll ans = 1;
    for (; p; p >>= 1, a = a * a % mod)
        if (p & 1)
            ans = ans * a % mod;
    return ans;
}
 
namespace hash {
    const int hash_mod = 1999993;
    const int MaxNode = 1e6 + 10;
 
    int point[hash_mod], nxt[MaxNode], v[MaxNode], idx[MaxNode], tot;
 
    inline void addGraph(int value, int myidx) {
        int hv = value % hash_mod;
        tot++;
        nxt[tot] = point[hv]; point[hv] = tot; v[tot] = value; idx[tot] = myidx;
    }
 
    inline int find(int value) {
        int hv = value % hash_mod;
        for (int tmp = point[hv]; tmp; tmp = nxt[tmp])
            if (value == v[tmp])
                return idx[tmp];
        return -1;
    }
};
 
namespace mat {
    const int MaxG = 1200;
    const int MaxC = 1e6 + 100;
    const int MaxN = 20;
    ll a[MaxN][MaxN];
    int g[MaxG][MaxN][MaxN], way[MaxG], edge[MaxG], tot0 = 700, N = 12;
    int cnt[MaxC], invcnt[MaxC], from[MaxC][2], tot;
 
    inline ll gauss(int n) {
        ll ans = 1, flag = 1;
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= n; j++) {
                while (a[i][j] < 0) a[i][j] += mod;
                a[i][j] %= mod;
            }
 
        for (int i = 1; i <= n; i++) {
            for (int j = i + 1; j <= n; j++) {
                while (a[j][i]) {
                    ll rate = a[i][i] / a[j][i];
                    for (int k = i; k <= n; k++)
                        a[i][k] = (a[i][k] + mod - rate * a[j][k] % mod) % mod;
                    swap(a[j], a[i]);
                    flag *= -1;
                }
            }
            if (!a[i][i]) return 0;
            ans = ans * a[i][i] % mod;
        }
 
        if (flag == -1)
            ans = mod - ans;
        return ans;
    }
 
    inline void makeG() {
        for (int t = 1; t <= tot0; t++) {
            memset(a, 0, sizeof(a));
            for (int i = 1; i < N; i++)
                for (int j = i + 1; j <= N; j++) {
                    if (getRand() < 0.8) {
                        a[i][i]++; a[j][j]++;
                        a[i][j] = a[j][i] = -1;
                        g[t][i][j] = g[t][j][i] = 1;   
                        edge[t]++;
                    }
                }
            way[t] = gauss(N - 1);
        }
 
        for (int i = 1; i <= tot0; i++)
            for (int j = 1; j <= tot0; j++) {
                int tmp = (ll)way[i] * way[j] % mod;
                if (tmp) {
                    ++tot;
                    cnt[tot] = tmp;
                    invcnt[tot] = qE(cnt[tot], mod - 2);
                    from[tot][0] = i; from[tot][1] = j;
                    hash::addGraph(cnt[tot], tot);
                }
            }
    }
 
    inline bool solve(int x) {
        for (int i = 1; i <= tot; i++) {
            int tar = (ll)x * invcnt[i] % mod;
            int idx = hash::find(tar);
            if (idx != -1) {
                int totedge = edge[from[i][0]] + edge[from[i][1]]
                            + edge[from[idx][0]] + edge[from[idx][1]];
                printf("%d %d\n", N * 4, totedge + 3);
 
                for (int wh = 0; wh < 1; wh++) {
                    for (int k = 1; k <= N; k++)
                        for (int p = k + 1; p <= N; p++) {
                            int a = from[i][0];
                            int b = from[i][1];
                            int c = from[idx][0];
                            int d = from[idx][1];
                            if (g[a][k][p])
                                printf("%d %d\n", k, p);
                            if (g[b][k][p])
                                printf("%d %d\n", k + N, p + N);
                            if (g[c][k][p])
                                printf("%d %d\n", k + 2 * N, p + 2 * N);
                            if (g[d][k][p])
                                printf("%d %d\n", k + 3 * N, p + 3 * N);
                        }
                }
 
                printf("%d %d\n", N, N * 2);
                printf("%d %d\n", N * 2, N * 3);
                printf("%d %d\n", N * 3, N * 4);
                //printf("\n");
 
                return true;
            }
        }
 
        return false;
    }
};
 
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
 
int main() {
    srand(1813319);
    mat::makeG();
 
    int T = getnum();
    while (T--) {
        int x = getnum();
        if (!x) {
            printf("3 1\n");
            printf("1 2\n");
            continue;
        }
 
        if (!mat::solve(x))
            printf("QwQ\n");
    }
}
```

C
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
#include <vector>
#include <bitset>
#include <queue>
using namespace std;
 
typedef long long ll;
const int mod = 998244353;
const int MaxN = 3100;
 
int cm[MaxN];
int g[MaxN][MaxN], n, deg[MaxN];
bitset<MaxN> from[MaxN];
 
inline void update(int &a, int b) {
    a += b;
    if (a >= mod) a -= mod;
}
 
int main() {
    scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        getchar();
        for (int j = 1; j <= n; j++) {
            char c = getchar();
            if (i != j && c == '0')
                g[i][j] = 1, deg[i]++;
        }
    }
 
    static int q[MaxN], tail = 0;
    for (int i = 1; i <= n; i++)
        if (!deg[i]) q[++tail] = i;
    for (int i = 1; i <= tail; i++) {
        int now = q[i];
        for (int j = 1; j <= n; j++)
            if (g[j][now]) {
                deg[j]--;
                if (!deg[j]) q[++tail] = j;
            }
    }
 
    cm[0] = 1;
    for (int i = 1; i <= n; i++)
        cm[i] = (ll)cm[i - 1] * 2 % mod;
 
    int ansTime = 0, ansKill = 0;
 
    for (int i = tail; i >= 1; i--) {
        int now = q[i];
        from[now][now] = 1;
        int cnt = from[now].count();
        update(ansTime, (ll)(cm[cnt] - 1) * cm[tail - cnt] % mod);
        update(ansKill, cm[tail - cnt]);
 
        for (int j = 1; j <= n; j++)
            if (g[now][j])
                from[j] |= from[now];
    }
 
    printf("%d %d\n", ansTime, ansKill);
}                 
``` 
