title: "[Solution][SPOJ LCS2]Longest Common Substring II"
date: 2015-01-23 14:18:45
tags: [SPOJ,后缀自动机]
categories: 题解
---
卡Hash卡Hash
<!--more-->
## 题目描述
[传送门](http://www.spoj.com/problems/LCS2/)

## 分析
做法类似[SPOJ LCS](http://www.spoj.com/problems/LCS/)。首先先对一个字符串建`SAM`，然后其余各个串在`SAM`上跑一遍，求出 每个串在`SAM`每个状态上的最长匹配长度。注意，如果在一个状态$S$上求得最大匹配长度为$Len$，那么它的所有$Parent$的最大匹配长度都可以被更新为$max(Parent)$。最后求一下每个状态各个串匹配长度中的最小值，再从所有状态中找出最大值即可。

然后我`T`了？？

刚开始以为是因为拓扑排序被卡常数了（拓扑排序也有常数？），然后改用按长度计数排序。还是`T`。结果最后发现……

我在循环里用了`strlen`……
我在循环里用了`strlen`……
我在循环里用了`strlen`……

瞬间$O(n)$变$O(n^2)$系列……

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <queue>
#include <vector>
using namespace std;
const int MAXN = 1e5 + 10;
const int MAXNODE = 2e5 + 10;
const int inf = 1e9;
struct samnode {
    samnode *par, *ch[26];
    int val, allmin, curmax, deg;
    samnode() {
        par = 0; memset(ch, 0, sizeof(ch));
        val = curmax = 0; allmin = inf;
        deg = 0;
    }
} node[MAXNODE], *root, *last;
queue<samnode*> q;
int order[MAXNODE], cnt[MAXNODE];
char s[20][MAXN];
int n, tot = 0;

inline void init() {
    root = last = &node[0];
}
inline void add(int c) {
    samnode *p = last;
    samnode *np = &node[++tot]; np->val = p->val + 1;
    while (p && !p->ch[c])
        p->ch[c] = np, p = p->par;
    if (!p)
        np->par = root;
    else {
        samnode *q = p->ch[c];
        if (q->val == p->val + 1)
            np->par = q;
        else {
            samnode *nq = &node[++tot]; nq->val = p->val + 1;
            memcpy(nq->ch, q->ch, sizeof(q->ch));
            nq->par = q->par;
            q->par = np->par = nq;
            while (p && p->ch[c] == q)
                p->ch[c] = nq, p = p->par;
        }
    } last = np;
}
inline void addstring(char *s) {
    for (int i = 0; i <= tot; i++) node[i].curmax = 0;
    int len = strlen(s), pilen = 0;
    samnode *now = root;
    for (int i = 0; i < len; i++) {
        int cur = s[i] - 'a';
        if (now->ch[cur])
            pilen++, now = now->ch[cur]; 
        else {
            while (now && !now->ch[cur])
                now = now->par;
            if (!now) 
                pilen = 0, now = root;
            else
                pilen = now->val + 1, now = now->ch[cur];
        }
        now->curmax = max(now->curmax, pilen);
    }
    for (int i = tot; i >= 0; i--) {
        samnode *now = &node[order[i]];
        now->allmin = min(now->allmin, now->curmax);
        if (!now->par) continue;
        if (now->curmax)
            now->par->curmax = max(now->par->curmax, now->par->val);
    }
}
int main() {
    init();
    while (scanf("%s", s[++n]) != EOF); n--;
    int len1 = strlen(s[1]);
    for (int i = 0; i < len1; i++)
        add(s[1][i] - 'a'); 
    for (int i = 0; i <= tot; i++) cnt[node[i].val]++;
    for (int i = 1; i <= tot; i++) cnt[i] += cnt[i - 1];
    for (int i = 0; i <= tot; i++) order[--cnt[node[i].val]] = i;
    for (int i = 2; i <= n; i++)
        addstring(s[i]);
    int ans = 0;
    for (int i = 0; i <= tot; i++)
        ans = max(ans, node[i].allmin);
    printf("%d\n", ans);
}
```