title: "[Solution][BZOJ2820]YY的GCD"
date: 2015-01-02 14:23:31
tags: [BZOJ,莫比乌斯反演,线性筛]
categories: 题解
---
神脑洞啊
<!--more-->
## 题目
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=2820)

## 分析
前面做个几个经典的题，要求的是$ \sum\_{1\leq i\leq A, 1\leq j\leq B}{[gcd(i,j)=k]} $，其中$k$,$A$,$B$为常数。我们可以将式子转化为$ \sum\_{1\leq i\leq min(A/k,B/k)}{\mu(i)\lfloor A/ki \rfloor \lfloor B/ki  \rfloor} $，然后在$O(\sqrt n)$的时间内求解。

详见[BZOJ2301](http://www.lydsy.com/JudgeOnline/problem.php?id=2301)，这里是[题解](http://gaotianyu1350.gitcafe.com/2014/12/30/BZOJ2301-ProblemB/)

但是这里的问题变成了$ \sum\_{1\leq i\leq A, 1\leq j\leq B}{[gcd(i,j)=p]} $，其中$p$为质数。变形一下得$ \sum\_{p为质数}{\sum\_{1\leq i\leq min(A/p,B/p)}{\mu(i)\lfloor  A/pi \rfloor \lfloor B/pi \rfloor}} $。

接下来利用的整除的一个性质：** 整除满足结合律，即$ A/x/y=A/(xy) $ **。令$X=pi$，则上式化为：
$$ \sum\_{1\leq X\leq min(A,B),p|X且p为质数}{\mu(X/p)\lfloor  A/X \rfloor \lfloor B/X \rfloor} $$

令$F(X)=\sum\_{p|X且p为质数}{\mu(X/p)}$，则最终答案
$$ Ans=\sum\_{1\leq X\leq min(A,B)}{F(x) \lfloor  A/X \rfloor \lfloor B/X \rfloor} $$

只要能求出$F(X)$一切就都解决了。令$g(X)$为$X$的所有质因子的次数的和，$h(x)$为质因子的个数，则
$$
F(X)=\begin{cases}
(-1)^{h(X)-1}h(X),& g(X)=h(X) \\\
(-1)^{h(X)},& g(X)=h(X)+1 \\\
0,& g(X)>h(X)+1
\end{cases}
$$
而$h(X)$和$g(X)$都可以由线性筛求出，至此问题完美解决。


## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
using namespace std;
typedef long long ll;
const int MAXN = 1e7 + 10;
const int M = 1e7;
bool check[MAXN] = {0};
int sum[MAXN] = {0}, cntc[MAXN] = {0}, cntz[MAXN] = {0}, prime[MAXN] = {0}, tot = 0;
ll qian[MAXN] = {0};
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
inline void makeMu(int M) {
    cntc[1] = cntz[1] = 0;
    for (int i = 2; i <= M; i++) {
        if (!check[i]) {
            prime[++tot] = i;
            cntc[i] = cntz[i] = 1;
        }
        for (int j = 1; j <= tot && prime[j] * i <= M; j++) {
            check[prime[j] * i] = tot;
            if (!(i % prime[j])) {
                cntc[i * prime[j]] = cntc[i] + 1;
                cntz[i * prime[j]] = cntz[i]; break;
            } else {
                cntc[i * prime[j]] = cntc[i] + 1;
                cntz[i * prime[j]] = cntz[i] + 1;
            }
        }
    }
}
inline void makeSum(int M) { sum[1] = 0;
    for (int i = 2; i <= M; i++) {
        if (cntz[i] == cntc[i]) sum[i] = ((cntz[i] - 1) % 2 ? -1 : 1) * cntz[i];
        else if (cntz[i] + 1 == cntc[i]) sum[i] = cntz[i] % 2 ? -1 : 1;
        else sum[i] = 0;
        qian[i] = qian[i - 1] + sum[i];
    }
}
int main() {
    makeMu(M); makeSum(M);
    int t = getnum();
    while (t--) {
        int n, m; n = getnum(); m = getnum();
        ll ans = 0; int tar = min(n, m);
        for (int i = 1; i <= tar; i++) {
            int cnt1 = n / i, cnt2 = m / i;
            int nxt = min(n / cnt1, m / cnt2);
            ans += (qian[nxt] - qian[i - 1]) * (ll)cnt1 * cnt2; i = nxt;
        }
        printf("%lld\n", ans);
    }
}
```