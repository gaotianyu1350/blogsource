title: "[Solution][POJ2154]Color"
date: 2015-03-07 07:15:37
tags: [POJ,欧拉函数,Burnside引理,Polya定理,群论]
categories: 题解
---
## 题目描述
[传送门](http://poj.org/problem?id=2154)

## 分析
首先这是一道群论题，要用到`Burnside引理`（或者是`Polya定理`）。这样的话我们需要求出对于所有置换情况的不动点的数目，暴力枚举显然不可行。

来看一看有什么规律。对于一个环来说，如果长度为$n$，位移的长度为$i$，那么一个循环的长度为$\frac{n}{gcd(n,i)}$，循环节的数量为$gcd(n,i)$。答案为$$\frac{\sum\_{i=1}^{i<n}n^{gcd(i,n)}}{n}$$
也就是
$$ \sum\_{i=1}^{i<n}n^{gcd(i,n)-1} $$

枚举$i$显然还是不行。那我们可以尝试枚举$gcd(i,n)$，令$a=n/gcd(i,n),t=i/gcd(i,n)$，则$n=a·gcd(i,n),i=t·gcd(i,n)$。如果一个$i$满足条件，必然要求$(a,t)=1$。也就是说满足条件的$i$的个数为$\phi(a)$（$t\leq a$）。枚举$gcd(i,n)$的时候还可以把$a$和$gcd(i,n)$反过来用，这样就只需要枚举到$\sqrt n$。注意特殊处理$(\sqrt n)^2=n$的情况

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

int prime[MaxN], totPrime = 0;
bool check[MaxN];

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

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

inline int qE(int a, int p, int mod) {
    int ans = 1;
    a %= mod;
    for (; p; p >>= 1, a = a * a % mod)
        if (p & 1)
            ans = ans * a % mod;
    return ans;
}

inline int phi(int x) {
    if (x == 1) return 1;
    int res = x;
    for (int i = 1; prime[i] * prime[i] <= x; i++)
        if (x % prime[i] == 0) {
            res = res / prime[i] * (prime[i] - 1);
            while (x % prime[i] == 0)
                x /= prime[i];
        }
    if (x > 1)
        res = res / x * (x - 1);
    return res;
}

int main() {
    makePrime(1e5);
    int tc = getnum();
    while (tc--) {
        int n, p;
        n = getnum(); p = getnum();
        
        int ans = 0, i;
        for (i = 1; i * i < n; i++) {
            if (n % i) continue;
            ans = (ans + phi(i) % p * qE(n, n / i - 1, p) % p) % p;
            ans = (ans + phi(n / i) % p * qE(n, i - 1, p) % p) % p;
        }
        if (i * i == n)
            ans = (ans + phi(i) % p * qE(n, i - 1, p) % p) % p;
        printf("%d\n", ans);
    }
}
```