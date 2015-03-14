title: "[Solution][SPOJ SUBLEX]Lexicographical Substring Search"
date: 2015-01-22 20:47:31
tags: [SPOJ,后缀自动机]
categories: 题解
---
自动机大法好
<!--more-->
## 题目描述
[传送门](http://www.spoj.com/problems/SUBLEX/)

## 分析
首先根据`后缀自动机`的性质，所有的相同子串在自动机上有且仅有一个状态与之对应。那么我们先建`后缀自动机`，然后从根开始`DFS`一遍，求出每个点下面能到达的状态有多少种。然后处理询问即可。

注意一些细节，比如根的一些特殊处理。

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
using namespace std;
const int MAXN = 1e5;
const int MAXNODE = 2e5;
struct samnode {
    samnode *par, *ch[26];
    int val, cnt; bool vis;
    samnode() {
        par = 0; memset(ch, 0, sizeof(ch));
        val = cnt = 0; vis = false;
    }
} node[MAXNODE], *root, *last;
int size;
inline void init() {
    root = last = &node[0];
    size = 0;
}
char s[MAXN], ans[MAXN]; int anslen;
inline void add(int c) {
    samnode *p = last;
    samnode *np = &node[++size]; np->val = p->val + 1;
    while (p && !p->ch[c])
        p->ch[c] = np, p = p->par;
    if (!p)
        np->par = root;
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
    }
    last = np;
}
void dfs(samnode *now) {
    now->vis = true;
    if (now == root) now->cnt = 0;
    else now->cnt = 1; 
    for (int i = 0; i < 26; i++)
        if (now->ch[i]) {
            if (!now->ch[i]->vis) dfs(now->ch[i]);
            now->cnt += now->ch[i]->cnt;
        }
}
void query(samnode *now, int k) {
    if (now != root && k == 1) return; 
    if (now != root) k--;
    for (int i = 0; i < 26; i++)
        if (now->ch[i]) {
            if (k <= now->ch[i]->cnt) {
                ans[anslen++] = i + 'a', query(now->ch[i], k);
                return;
            }
            else k -= now->ch[i]->cnt;
        }
}
int main() {
    scanf("%s", s); int len = strlen(s);
    init();
    for (int i = 0; i < len; i++) add(s[i] - 'a');
    dfs(root);
    int q; scanf("%d", &q);
    for (int i = 1; i <= q; i++) {
        int x; scanf("%d", &x);
        anslen = 0;
        query(root, x); ans[anslen] = '\0';
        printf("%s\n", ans);
    }
}
```