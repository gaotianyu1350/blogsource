title: "[Solution][BZOJ2301][HAOI2011]Problem b"
date: 2014-12-30 20:06:44
tags: [BZOJ,数论,莫比乌斯反演,gcd]
categories: 题解
---
技巧小优化
<!--more-->
## 题目
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=2301)

## 分析
主要思路和[BZOJ2045](http://www.lydsy.com/JudgeOnline/problem.php?id=2045)一样，这里是[题解](http://gaotianyu1350.gitcafe.com/2014/12/29/BZOJ2045-%E5%8F%8C%E4%BA%B2%E6%95%B0/)。

和`BZOJ2045`不同的地方是，这次不仅有上界，还有下界。

令$solve(l,r,k)=\sum\_{1\leq i\leq l,1\leq j\leq r}{[gcd(i,j)=k]}$，利用容斥原理，可以将答案转化成

$$ solve(b,d,k)-solve(b,c-1,k)-solve(a-1,d,k)+solve(a-1,c-1,k) $$

通过`BZOJ2045`我们已经知道$solve(a,b,k)=\sum\_{1\leq i\leq min(\frac{a}{k},\frac{b}{k})}{\mu(i)\lfloor \frac{a}{ik} \rfloor \lfloor \frac{b}{ik} \rfloor} $，但是这样求的话会`TLE`，怎么办呢？

很多有关`除法`的问题都有一个这样的优化，$A$除以一定连续区间的数的答案是不变的。而且不同的答案一共只有$2\sqrt A$个（证明？似乎可以用反比例函数的导数）。假如当前的数字是$i$，除法结果为$A/i$（此处和以下$/$均代表整除），那么答案同为$A/i$的最大的$i$为$A/(A/i)$。

那两个数$A$和$B$呢？是不是结果的组合个数变成了$4\sqrt A \sqrt B$？当然不是。每次相同答案的区间其实取决于是$A/(A/i)$小还是$B/(B/i)$小，所以最后答案个数的组合仍是根号级别的。

这样我们求出$\mu$和它的前缀和，再在计算的时候采用以上的优化，问题就完美解决了。

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
const int MAXN = 5 * 1e4 + 10;
const int M = 5 * 1e4;
int prime[MAXN] = {0}, mu[MAXN] = {0}, sum[MAXN] = {0}, tot = 0;
bool check[MAXN] = {0};
inline void makeMu() { mu[1] = 1;
    for (int i = 2; i <= M; i++) {
        if (!check[i]) {
            prime[++tot] = i;
            mu[i] = -1;
        }
        for (int j = 1; j <= tot && prime[j] * i <= M; j++) {
            check[prime[j] * i] = true;
            if (i % prime[j] == 0) {
                mu[i * prime[j]] = 0;
                break;
            } else mu[i * prime[j]] = -mu[i];
        }
    }
    for (int i = 1; i <= M; i++) sum[i] = sum[i - 1] + mu[i];
}
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);    
}
inline ll solve(int a, int b, int k) {
    a /= k, b /= k;
    ll ans = 0; int tar = min(a, b);
    for (int i = 1; i <= tar; i++) {
        ll cnt1 = a / i; ll tar1 = a / cnt1;
        ll cnt2 = b / i; ll tar2 = b / cnt2;
        ll nxt = min(tar1, tar2);
        ans += cnt1 * cnt2 * (sum[nxt] - sum[i - 1]); i = nxt;
    } return ans;
}
int main() { makeMu();
    int n, a, b, c, d, k;
    n = getnum();
    for (int i = 1; i <= n; i++) {
        a = getnum(); b = getnum(); c = getnum(); d = getnum(); k = getnum();
        printf("%lld\n", solve(b, d, k) - solve(b, c - 1, k) - solve(a - 1, d, k) + solve(a - 1, c - 1, k));
    }
}
```