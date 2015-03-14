title: "[Solution][SPOJ LCS]Longest Common Substring"
date: 2015-01-22 21:47:41
tags: [SPOJ,后缀自动机,AC自动机]
categories: 题解
---
后缀、AC自动机经典应用
<!--more-->
## 题目描述
[传送门](http://www.spoj.com/problems/LCS/)

## 分析
`AC自动机`的做法就不说了

`后缀自动机`的话，先对$A$串建立$SAM$，然后将对$B$串的字符一个一个匹配。设当前处在自动机上的状态$S$，匹配长度$len$，接下来待匹配的字符为$x$。如果`s->ch[x]!=null`，就直接前往下一个状态，且`len++`。否则，找到$s$的$parent$里面第一个$ch_x$不为空的，根据`后缀自动机`的性质，这一定是全部状态中$max(s)$最大的，其实这一步相当于`AC自动机`上走失配边。我们将$s$更新为这个$parent$的$ch_x$，$len$更新为这个$parent$的$max(s)+1$。如果没有这样一个$parent$，那令`S = root, len = 0`。

# 代码
```
#include <cstdio>
#include <cstring>
#include <cstdlib>
#include <cmath>
#include <iostream>
#include <algorithm>
using namespace std;
const int MAXN = 2.5e5 + 100;
const int MAXNODE = 5e5 + 100;
struct samnode {
    samnode *par, *ch[26];
    int val;
    samnode() { 
        par = 0; memset(ch, 0, sizeof(ch));
        val = 0;
    }
} node[MAXNODE], *root, *last;
int size = 0;
char s1[MAXN], s2[MAXN];
inline void init() {
    last = root = &node[0];
}
inline void add(int c) {
    samnode *p = last;
    samnode *np = &node[++size]; np->val = p->val + 1;
    while (p && !p->ch[c])
        p->ch[c] = np, p = p->par;
    if (!p) np->par = root;
    else {
        samnode *q = p->ch[c];
        if (q->val == p->val + 1)
            np->par = q;
        else {
            samnode *nq = &node[++size]; nq->val = p->val + 1;
            memcpy(nq->ch, q->ch, sizeof(q->ch));
            nq->par = q->par;
            q->par = np->par = nq;
            while (p && p->ch[c] == q)
                p->ch[c] = nq, p = p->par;
        }
    } last = np;
}
int main() {
    init();
    scanf("%s%s", s1, s2); int len1 = strlen(s1), len2 = strlen(s2);
    for (int i = 0; i < len1; i++)
        add(s1[i] - 'a');
    int ans = 0, curlen = 0; samnode *now = root;
    for (int i = 0; i < len2; i++) {
        int cur = s2[i] - 'a';
        if (now->ch[cur])
            now = now->ch[cur], curlen++; 
        else {
            while (now && !now->ch[cur]) now = now->par;
            if (!now)
                curlen = 0, now = root;
            else
                curlen = now->val + 1, now = now->ch[cur];
        }
        ans = max(ans, curlen);
    }
    printf("%d\n", ans);
}
```