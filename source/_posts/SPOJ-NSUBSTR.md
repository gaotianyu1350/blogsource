title: "[Solution][SPOJ NSUBSTR]Substrings"
date: 2015-01-22 20:46:18
tags: [SPOJ,后缀自动机]
categories: 题解
---
后缀自动机第一题
<!--more-->
## 题目描述
[传送门](http://www.spoj.com/problems/NSUBSTR/)

## 分析
令$f_i$为长度为$i$的子串出现的最多的次数。首先建立`后缀自动机`，对于每一个节点$s$，假设控制的子串长度为$[min(s),max(s)]$，$Right$集合个数为$r$，那么它可以去更新$f\_{max(s)}=max(f\_{max(s)},r)$。但是最后不要忘记用$f\_i$去更新$f\_{i-1}$。

如何求出$r$？注意从$root$给定按照字符串节点走到$last$节点经过的所有状态的$r$都是$1$。其余状态的$r$为所有儿子的和，用拓扑排序求解即可。

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <queue>
using namespace std;
const int MAXN = 2.5e5 + 10;
const int MAXNODE = 5e5 + 10;
struct samnode {
    samnode *par, *ch[26];
    int val, r, deg;
    samnode() { 
        memset(ch, 0, sizeof(ch)); par = 0; val = r = deg = 0;
    }
}node[MAXNODE], *last, *root; int size = 0;
queue<samnode*> q;
char s[MAXN];
int f[MAXN];
inline void add(int c) {
    samnode *p = last;
    samnode *np = &node[++size]; np->val = p->val + 1;
    while (p && !p->ch[c])
        p->ch[c] = np, p = p->par;
    if (!p) 
        np->par = root;
    else {
        samnode *q = p->ch[c];
        if (p->val + 1 == q->val) 
            np->par = q;
        else {
            samnode *nq = &node[++size]; nq->val = p->val + 1;
            memcpy(nq->ch, q->ch, sizeof(q->ch));
            nq->par = q->par;
            q->par = np->par = nq;
            while (p && p->ch[c] == q)
                p->ch[c] = nq, p = p->par;
        }
    }
    last = np;
}
inline void init() { last = root = &node[0]; }
int main() {
    scanf("%s", s); init();
    int len = strlen(s);
    for (int i = 0; i < len; i++)
        add(s[i] - 'a');
    for (int i = 1; i <= size; i++)
        node[i].par->deg++;
    for (int i = 1; i <= size; i++)
        if (!node[i].deg) 
            q.push(&node[i]);
    samnode *now = root->ch[s[0] - 'a'];
    int cur = 1;
    while (cur < len) {
        now->r = 1; now = now->ch[s[cur++] - 'a'];
    } now->r = 1;
    while (!q.empty()) {
        samnode *now = q.front(); q.pop();
        if (!now->par) continue;
        f[now->val] = max(f[now->val], now->r);
        now->par->r += now->r; 
        now->par->deg--;
        if (!now->par->deg) q.push(now->par);
    }
    for (int i = len - 1; i >= 1; i--) f[i] = max(f[i], f[i + 1]);
    for (int i = 1; i <= len; i++) printf("%d\n", f[i]); 
}
```