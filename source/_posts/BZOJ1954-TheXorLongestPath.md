title: "[Solution][BZOJ1954][POJ3764]The xor-longest Path"
date: 2014-12-19 20:09:47
tags: [BZOJ,POJ,xor方程组]
categories: 题解
---
树上最短xor路径
<!--more-->
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1954)

## 分析
可以先求出$a[i]$表示$i$到根的路径的xor和。然后问题转变成从$n$个数里面选出两个数使得xor值最大。结果我刚开始当成任选数字求xor和最大，逗比的写了线性基。正解是用所有的数字构建一棵trie树，然后枚举第一个数，在trie树上找可以使得xor值最大的第二个数。

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
#include <queue>
using namespace std;

typedef long long ll;
const int MAXN = 1e5 + 10;
const int MAXBIT = 30;

int point[MAXN] = {0}, nxt[MAXN * 2] = {0}, v[MAXN * 2] = {0}, w[MAXN * 2] = {0}, tot = 0;
int a[MAXN] = {0}, father[MAXN] = {0}, ch[MAXN * MAXBIT][2] = {0}, size = 0, n;
queue<int> q;

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c -'0';
    return ans * (flag ? -1 : 1);
}

inline void addedge(int x, int y, int ww) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y; w[tot] = ww;
}

inline void Insert(int x) {
    int now = 0;
    for (int i = MAXBIT; i >= 0; i--) {
        int cur = (x & (1 << i)) ? 1 : 0;
        if (!ch[now][cur]) ch[now][cur] = ++size;
        now = ch[now][cur];
    }
}

int main() {
    n = getnum();
    int x, y, ww;
    for (int i = 1; i < n; i++) {
        x = getnum(); y = getnum(); ww = getnum();
        addedge(x, y, ww); addedge(y, x, ww);
    }
    while (!q.empty()) q.pop(); q.push(1); Insert(0);
    while (!q.empty()) {
        int now = q.front(); q.pop();
        for (int tmp = point[now]; tmp; tmp = nxt[tmp])
            if (v[tmp] != father[now]) {
                a[v[tmp]] = a[now] ^ w[tmp]; father[v[tmp]] = now;
                q.push(v[tmp]); Insert(a[v[tmp]]);
            }
    }
    int ans = 0;
    for (int i = 1; i <= n; i++) {
        int tmpans = a[i], now = 0;
        for (int j = MAXBIT; j >= 0; j--) {
            int cur = (tmpans & (1 << j)) ? 0 : 1;
            if (!ch[now][cur]) cur ^= 1;
            if (!ch[now][cur]) break;
            if (cur) tmpans ^= (1 << j); now = ch[now][cur];
        }
        ans = max(ans, tmpans);
    } 
    printf("%d\n", ans);
}
```