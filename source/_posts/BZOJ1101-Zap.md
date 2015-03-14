title: "[Solution][BZOJ1101][POI2007]Zap"
date: 2014-12-30 20:14:39
tags: [BZOJ,数论,莫比乌斯反演,gcd]
categories: 题解
---
经典题
<!--more-->
## 题目
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1101)

## 分析
同[BZOJ2301](http://www.lydsy.com/JudgeOnline/problem.php?id=2301)

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
    int n, a, b, k;
    n = getnum();
    for (int i = 1; i <= n; i++) {
        a = getnum(); b = getnum(); k = getnum();
        printf("%lld\n", solve(a, b, k));
    }
}
```