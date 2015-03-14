title: "[Solution][Cerc2007]robotic sort"
date: 2015-01-28 21:01:48
tags: [BZOJ,splay,Cerc]
categories: 题解
---
看错题了是什么RP
<!--more-->
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1552)

## 分析
数据结构裸题。用`Splay`也行`Treap`也行。结果刚开始看错题了。题目里说当编号相同的时候按照输入的原始次序操作。结果我看成了编号相同时按照当前次序操作。结果写的特别蛋疼。

再有就是有好几个小错误调试了半天。注意`pushdown()`和`update()`的时机。要先保证正确再尝试减小常数。还有就是数据结构题一定要对拍！

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
using namespace std;
const int inf = 1e9;
const int MaxN = 1e5 + 100;
struct splay_node {
    splay_node *pre, *ch[2], *minwh;
    int val, cnt, rev, loc;
    splay_node ();
    void update();
    void pushdown(); 
    void setch(splay_node *child, int wh) {
        pushdown();
        ch[wh] = child; child->pre = this;
        update();
    }
    int getwh() {
        return this == pre->ch[0] ? 0 : 1;
    }
} pool[MaxN], *root;
int n, a[MaxN], tot = 0;
bool sncmp(splay_node *a, splay_node *b) {
    return a->val < b->val || (a->val == b->val && a->loc < b->loc);
}
void splay_node::update() {
    cnt = 1 + ch[0]->cnt + ch[1]->cnt;
    minwh = this;
    if (sncmp(ch[0]->minwh, this))
        minwh = ch[0]->minwh;
    if (sncmp(ch[1]->minwh, minwh))
        minwh = ch[1]->minwh;
}
void splay_node::pushdown() {
    if (rev) {
        swap(ch[0], ch[1]);
        ch[0]->rev ^= 1;
        ch[1]->rev ^= 1;
        rev = 0;
    }
}
splay_node::splay_node() {
    pre = ch[0] = ch[1] = pool;
    minwh = this;
    val = inf;
    cnt = rev = loc = 0;
}
splay_node* build(int l, int r) {
    if (l > r) return pool;
    int mid = (l + r) >> 1;
    splay_node *now = pool + (++tot);
    now->val = a[mid]; now->loc = mid;
    now->cnt = 1;
    if (l < r) {
        now->setch(build(l, mid - 1), 0);
        now->setch(build(mid + 1, r), 1);
    }
    return now;
}
inline void rotate(splay_node *now) {
    splay_node *of = now->pre, *gf = now->pre->pre;
    gf->pushdown();
    of->pushdown();
    now->pushdown();
    int wh = now->getwh();
    of->setch(now->ch[wh ^ 1], wh);
    now->setch(of, wh ^ 1);
    now->pre = gf;
    if (gf != pool)
        gf->ch[gf->ch[0] == of ? 0 : 1] = now;
}
inline void splay(splay_node *now, splay_node *tar) {
    for (; now->pre != tar; rotate(now))
        if (now->pre->pre != tar) 
            now->getwh() == now->pre->getwh() ? rotate(now->pre) : rotate(now);
    if (tar == pool)
        root = now;
}
inline splay_node* getnxt(splay_node *now, splay_node *tar) {
    splay(now, tar);
    now->pushdown(); now = now->ch[1];
    if (now == pool) return pool;
    for (now->pushdown(); now->ch[0] != pool; now = now->ch[0], now->pushdown());
    return now;
}
inline splay_node* getkth(int k) {
    splay_node *now = root;
    while (now != pool) {
        now->pushdown();
        if (k <= now->ch[0]->cnt) now = now->ch[0];
        else if (now->ch[0]->cnt + 1 == k) break;
        else k -= now->ch[0]->cnt + 1, now = now->ch[1];
    } return now;
}
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
void print(splay_node *now) {
    putchar('(');
    now->pushdown();
    if (now->ch[0] != pool) print(now->ch[0]);
    printf("%d ", now->val);
    if (now->ch[1] != pool) print(now->ch[1]);
    putchar(')');
}
inline int getrank(splay_node *now) {
    splay(now, pool);
    return now->ch[0]->cnt + 1;
}
int main() {
    n = getnum();
    for (int i = 1; i <= n; i++)
        a[i] = getnum();
    root = build(1, n);
    for (int i = 1; i <= n; i++) {
        splay_node *minwh, *now = pool;
        if (i == 1)
            minwh = root->minwh;
        else {
            now = getkth(i - 1);
            splay(now, pool);
            now->pushdown();
            minwh = now->ch[1]->minwh;
        } printf("%d", getrank(minwh));
        if (i < n) putchar(' '); else putchar('\n');
        if (now != pool) splay(now, pool);
        minwh = getnxt(minwh, now);
        if (minwh != pool) 
            splay(minwh, now);
        if (now == pool) {
            if (minwh == pool)
                root->rev ^= 1;
            else 
                root->ch[0]->rev ^= 1;
        } else {
            root->ch[1]->pushdown();
            if (minwh == pool)
                root->ch[1]->rev ^= 1;
            else
                root->ch[1]->ch[0]->rev ^= 1;
        }
    }
}
```