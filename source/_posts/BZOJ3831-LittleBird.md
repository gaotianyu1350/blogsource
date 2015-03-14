title: "[Solution][BZOJ3831][POI2014]Little Bird"
date: 2015-01-27 16:19:43
tags: [BZOJ,POI,单调队列,DP]
categories: 题解
---
水题不会做
<!--more-->
## 题目分析
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3831)

## 分析
大体看上去就是一道单调队列的题目。但是我卡在了判断谁更优上面。令$f\_i$表示到$i$的最少体力消耗，$height\_i$表示$i$的高度。判断两个点谁更优秀不仅要看$f$还要看$height$。刚开始我以为有两个因素就没法决定谁更优了，但是其实是可以判断的。

如果$f\_i\lt f\_j$，$i$一定比$j$优秀。因为如果$height\_i\geq height\_j$就不用说了，如果$height\_i\lt height\_j$，$i$可以花$1$的代价跳到高的地方，顶多和$j$一样，不会比$j$差。

如果$f\_i=f\_j$，那当然就是谁的$height$高谁优秀。

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <deque>
using namespace std;
const int MaxN = 1e6 + 10;
const int MaxQ = 27;
int height[MaxN], f[MaxN], n, q;
struct MonoQueue { 
    deque<int> q;
    int k;
    void SetNew(int kk) {
        while (!q.empty()) q.pop\_back();
        k = kk;
    }
    bool cmp(int a, int b) {
        if (f[a] < f[b]) return true;
        else if (f[a] > f[b]) return false;
        else return height[a] > height[b];
    }
    void AddNew(int x) {
        while (!q.empty() && cmp(x, q.back())) q.pop\_back();
        q.push\_back(x);
    }
    int GetBest(int x) {
        while (!q.empty() && q.front() < x - k) q.pop\_front();
        return q.front();
    }
} mq;
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
inline int Solve(int k) {
    mq.SetNew(k);
    mq.AddNew(1);
    for (int i = 2; i <= n; i++) {
        int best = mq.GetBest(i);
        f[i] = f[best] + (height[i] >= height[best] ? 1 : 0);
        mq.AddNew(i);
    }
    return f[n];
}
int main() {
    n = getnum();
    for (int i = 1; i <= n; i++)
        height[i] = getnum();
    q = getnum();
    for (int i = 1; i <= q; i++) {
        int k = getnum();
        printf("%d\n", Solve(k));
    }
}
```
