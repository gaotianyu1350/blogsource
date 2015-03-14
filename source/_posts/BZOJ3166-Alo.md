title: "[Solution][BZOJ3166][HEOI2013]Alo"
date: 2014-12-21 08:14:46
tags: [BZOJ,HEOI,xor,Trie,set]
categories: 题解
---
Welcome to ALO
<!--more-->
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3166)

## 分析
想到了对于每个数作为次大的情况分别处理。但是如何确定每个数控制的范围呢？天啊我竟然没有想到要sort……

排序后怎样维护呢？我们需要快速的找到一个位置的前驱后继，还要支持插入。所以用set。对于一个数，它的控制范围是[前驱的前驱+1,后继-1]并上[前驱+1,后继的后继-1]。至于区间查询xor某数的最大值，交给可持久化trie就好。

这里是可持久化trie的一道题[题面](http://www.lydsy.com/JudgeOnline/problem.php?id=3261)和[题解](http://gaotianyu1350.gitcafe.com/2014/12/20/BZOJ3261-%E6%9C%80%E5%A4%A7%E5%BC%82%E6%88%96%E5%92%8C/)

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
#include <set>
using namespace std;
const int MAXN = 5 * 1e4 + 10;
const int MAXBIT = 30;
const int MAXNODE = 5 * 33 * 1e4 + 10;
struct num {
    int value, loc;
    bool operator < (const num other) const {
        return value < other.value;
    }
}a[MAXN];
int size[MAXNODE] = {0}, ch[MAXNODE][2], tot = 0;
int root[MAXN] = {0}, n;
set<int> S;
inline int getnum() {
    char c; int ans = 0; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
inline void add(int loc, int value) {
    int now = root[loc] = ++tot, old = root[loc - 1];
    for (int i = MAXBIT; i >= 0; i--) {
        size[now] = size[old] + 1; int cur = (value >> i & 1);
        ch[now][cur ^ 1] = ch[old][cur ^ 1]; now = ch[now][cur] = ++tot;
        old = ch[old][cur];
    } size[now] = size[old] + 1;
}
inline int query(int lNow, int lPre, int value) {
    int ans = 0, now = root[lNow], pre = root[lPre];
    for (int i = MAXBIT; i >= 0; i--) {
        int cur = (value >> i & 1) ^ 1;
        if (!(size[ch[now][cur]] - size[ch[pre][cur]])) cur ^= 1;
        else ans += (1 << i);
        now = ch[now][cur], pre = ch[pre][cur];
    } return ans;
}
int main() {
    n = getnum();
    for (int i = 1; i <= n; i++) a[i].value = getnum(), add(i, a[i].value), a[i].loc = i;
    sort(a + 1, a + 1 + n);
    S.clear(); S.insert(a[n].loc); S.insert(0); S.insert(n + 1);
    int ans = 0;
    for (int i = n - 1; i >= 1; i--) {
        set<int>::iterator p = S.insert(a[i].loc).first;
        set<int>::iterator pre = p, nxt = p; pre--; nxt++;
        if (pre != S.begin()) { 
            set<int>::iterator pp = pre; pp--;
            ans = max(ans, query(*nxt - 1, *pp, a[i].value));
        } 
        if (++nxt != S.end())
            ans = max(ans, query(*nxt - 1, *pre, a[i].value));
    }
    printf("%d\n", ans);
}
```