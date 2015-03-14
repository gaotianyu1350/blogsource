title: "[Solution][BZOJ2844]albus就是要第一个出场"
date: 2014-12-19 19:23:04
tags: [BZOJ,高斯消元,xor方程组]
categories: 题解
---
LaaLLaaL又因为爆int犯傻了。。。
<!--more-->
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=2844)

## 分析
首先要求线性基(linear base)。什么是线性基？类似向量里面的基底，用线性基里面的数可以表示出所有原来的集合里数的xor值，而且线性基里面的数不能被别的数表示。说白了就是把数拆成二进制形式后进行高斯消元得到的矩阵。线性基有一个非常简便的方法可以求出，效率$O(60n)$。
```
inline void calcLinearBase() {
    for (int i = 1; i <= n; i++)
        for (int j = MAXBIT; j >= 0; j--)
            if (a[i] & (1 << j)) {
                if (!lb[j]) { lb[j] = a[i]; break; }
                else a[i] ^= lb[j];
            }
}
```
求完线性基以后，求最大xor值，排名第$k$的xor值，xor值$S$排名第几都可以做到。这道题是求特定值的排名。注意，假设线性基中有$tot$个数，也就是说有$n-tot$个数在消元中消失了，那么对于每一个xor值（每个xor值有且只有一种用线性基表示的方法），都有$2^{n-tot}$种方式得到（因为那些消为$0$的数可选可不选）。
要及时取模，小心爆int！

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
const int MAXN = 1e5 + 10;
const int MAXBIT = 30;
const int MOD = 10086;
int a[MAXN], lb[MAXBIT + 10] = {0}, loc[MAXBIT + 10] = {0}, tot = 0;
int n, tar;
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
inline void calcLinearBase() {
    for (int i = 1; i <= n; i++)
        for (int j = MAXBIT; j >= 0; j--)
            if (a[i] & (1 << j)) {
                if (!lb[j]) { lb[j] = a[i]; break; }
                else a[i] ^= lb[j];
            }
    for (int i = 0; i <= MAXBIT; i++)
        if (lb[i]) loc[i] = tot++; 
}
inline int quickpow(int a, int p) {
    int ans = 1;
    for (; p; a = a * a % MOD, p >>= 1)
        if (p & 1) ans = ans * a % MOD;
    return ans;
}
int main() {
    n = getnum();
    for (int i = 1; i <= n; i++) a[i] = getnum();
    int tmp = tar = getnum();
    calcLinearBase();
    int ans = 0;
    for (int i = MAXBIT; i >= 0; i--) 
        if (lb[i]) {
            if (tmp & (1 << i)) tmp ^= lb[i];
            if (tar & (1 << i)) ans += (1 << loc[i]);
        }
    ans++;
    ans = (quickpow(2, n - tot) * ((ans - 1) % MOD) + 1) % MOD;
    printf("%d\n", ans);
}
```