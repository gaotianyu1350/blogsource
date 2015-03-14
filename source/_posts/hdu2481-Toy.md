title: "[Solution][HDU2481]Toy"
date: 2015-03-07 14:42:26
tags: [HDU,Burnside引理,Polya定理,欧拉函数,递推,矩阵乘法,快速乘]
categories: 题解
---
## 题目描述
[传送门](http://acm.hdu.edu.cn/showproblem.php?pid=2481)

## 分析
这道题不需要考虑旋转的情况和[BZOJ1002轮状病毒](http://www.lydsy.com/JudgeOnline/problem.php?id=1002)是相同的。记得当时做的时候完全不明白式子是怎么推出来的……今天看了几位大神的题解，来尝试着推一下。

先在环上取两个固定的点$A$和$B$，令$f\_i$表示环上有$i$个点，$A$和$B$不相连的方案数，令$g\_i$表示环上有$i$个点，$A$和$B$相连的方案数。答案显然为$f\_n+g\_n$。

考虑$f\_i$，$K$点是$A$所在的联通块的最靠右边的一个点，联通块一共有$n-k$个点的话，方案数为$(n-k)f\_k$（如图一）。那么可得：
$$ f\_n=\sum\_{k=0}^{k< n}(n-k)f\_k $$
其中$f\_0=1$。

再考虑$g\_i$，如果$A$和$B$所在的联通块的长度为$n-k$，$A$和$B$在其中的位置共有$(n-k-1)$种选择，连向中心点的边有$(n-k)$种选择，所以方案数为$(n-k)(n-k-1)f\_k$（如图二）。那么可得：
$$ g\_n=\sum\_{k=0}^{k<n-1}(n-k)(n-k-1)f\_k $$
其中$g\_0=g\_1=0$

![图一](/img/hdu2481/a.jpg)
![图二](/img/hdu2481/b.jpg)

接下来我们进行一些变形。

先令$s\_n=\sum\_{i=0}^{i\leq n}f\_i$

$$
\begin{align}
f\_n=& s\_{n-1}+f\_{n-1}\\\
=& 2f\_{n-1}+s\_{n-2}\\\
=& 2f\_{n-1}+(f\_{n-1}-f\_{n-2})\\\
=& 3f\_{n-1}-f\_{n-2}
\end{align}
$$

$$
\begin{align}
g\_n=&g\_{n-1}+2f\_{n-2}+4f\_{n-3}+...2(n-1)f\_0 \\\
=& g\_{n-1}+2f\_{n-1}\\\
=& 2(s\_{n-1}-f\_0)\\\
=& 2(f\_n-f\_{n-1}-1)
\end{align}
$$

因为$f$的递推式是线性递推关系，所以用矩阵乘法就可以算出来，这样$g$的值也就能算出来了。

算出不加旋转的方案数，我们就可以考虑加旋转特技了。特技部分和[POJ2154](http://poj.org/problem?id=2154)类似，这里是[题解](http://gaotianyu1350.gitcafe.io/2015/03/07/POJ2154-Color/)。

唯一不同的是，这里颜色不能随便选。我们可以把颜色理解成每个点是如何连接的。显然对于一个不动点来说，每个循环里面点的连接方式完全一样，如果循环的个数为$i$，则连接的方案数为$f\_i+g\_i$。

还要特别注意取模和除法的问题。除法可以这样解决：
$$ a \% b / p = a \% bp / p \% b $$
这样的话模数会非常大，乘法会爆`long long`，所以乘法要用类似快速幂的快速乘。下面是黑科技——$O(1)$快速乘：
```c++  
inline ll mol(ll a, ll b) {
    return ((a * b - (ll)(((long double)a * b + 0.5) / mod) * mod) % mod + mod) % mod;
}
```
原理还是很好理解的……

<!--more-->
## 代码
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
using namespace std;

typedef long long ll;
const int MaxN = 1e5 + 100;

ll mod;
int prime[MaxN], totPrime = 0;
bool check[MaxN];

inline ll mol(ll a, ll b) {
    ll res = 0;
    for (; b; b >>= 1, a = a * 2 % mod)
        if (b & 1)
            res = (res + a) % mod;
    return res;
}

inline void update(ll &a, ll b) {
    a = (a + b) % mod;
}

namespace mat{
    struct matrix {
        ll a[5][5];
        int n, m;
        matrix(int nn, int mm) {
            n = nn, m = mm;
            memset(a, 0, sizeof(a));
        }
        void set0() {
            memset(a, 0, sizeof(a));
            for (int i = 1; i <= n; i++) a[i][i] = 1;
        }
        matrix operator = (const matrix &o) {
            memcpy(a, o.a, sizeof(o.a));
            n = o.n, m = o.m;
            return *this;
        }
        matrix operator * (const matrix &o) const {
            matrix ans(n, o.m);
            for (int i = 1; i <= ans.n; i++)
                for (int j = 1; j <= ans.m; j++)
                    for (int k = 1; k <= m; k++)
                        update(ans.a[i][j], mol(a[i][k], o.a[k][j]));
            return ans;
        }
        matrix operator ^ (ll p) const {
            if (n != m) return *this;
            matrix ans(n, n), tmp(n, n);
            tmp = *this; ans.set0();
            for (; p; p >>= 1, tmp = tmp * tmp)
                if (p & 1)
                    ans = ans * tmp;
            return ans;
        }
    };

    inline ll solve(int n) {
        if (n == 1) return 1 % mod;
        if (n == 2) return 5 % mod;

        matrix trans(3, 3);
        trans.a[1][2] = 1;
        trans.a[2][1] = mod - 1, trans.a[2][2] = 3, trans.a[2][3] = 2;
        trans.a[3][3] = 1;

        matrix ans(3, 1);
        ans.a[1][1] = 1, ans.a[2][1] = 5, ans.a[3][1] = 1;

        trans = trans ^ (n - 2);
        ans = trans * ans;
        return ans.a[2][1];
    }
};

inline void makePrime(int M) {
    for (int i = 2; i <= M; i++) {
        if (!check[i])
            prime[++totPrime] = i;
        for (int j = 1; j <= totPrime && prime[j] * i <= M; j++) {
            check[prime[j] * i] = true;
            if (i % prime[j] == 0)
                break;
        }
    }
}

inline int phi(int n) {
    if (n == 1) return 1;
    int res = n;
    for (int i = 1; prime[i] * prime[i] <= n; i++)
        if (n % prime[i] == 0) {
            res = res / prime[i] * (prime[i] - 1);
            while (n % prime[i] == 0)
                n /= prime[i];
        }
    if (n > 1)
        res = res / n * (n - 1);
    return res;
}

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while (((c = getchar()) == ' ' || c == '\n' || c == '\r') && c != -1);
    if (c == -1) return -1;
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

int main() {
    makePrime(1e5);

    while (1) {
        int n, m;
        n = getnum(), m = getnum();
        if (n == -1) break;

        mod = (ll)m * n;
        ll ans = 0; int i;
        for (i = 1; i * i < n; i++)
            if (n % i == 0) {
                //ans += phi(n / i) * mat::solve(i) / n;
                //ans += phi(i) * mat::solve(n / i) / n;
                update(ans, mol(phi(n / i) % mod, mat::solve(i)));
                update(ans, mol(phi(i) % mod, mat::solve(n / i)));
            }
        if (i * i == n)
            //ans += phi(i) * mat::solve(i) / n;
            update(ans, mol(phi(i) % mod, mat::solve(i)));
        ans = ans / n % m;
        printf("%d\n", (int)ans);
    }
}
```