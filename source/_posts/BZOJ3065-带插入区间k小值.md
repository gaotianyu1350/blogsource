title: "[Solution][BZOJ3065]带插入区间K小值"
date: 2015-03-05 15:49:56
tags: [替罪羊树,可持久化线段树,垃圾回收,树套树]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3065)

## 分析
丧心病狂数据结构题，解法多种多样。我写的是替罪羊树套可持久化线段树。

替罪羊树是一种无须旋转的平衡树，删除查询$log(n)$，插入均摊$log(n)$。它的原理基于“部分重建”，也就是当平衡树不再平衡的时候进行重建。

对于每个替罪羊树上的节点，我们维护一棵可持久化线段树，来维护这个节点控制的区间内的数。如何合并？这里用到了主席讲的线段树的合并。

查询的时候，先采用类似线段树查询的方法确定查询区间覆盖的替罪羊树上的节点，然后用类似主席树查询的方法查询。

不过关于替罪羊树我有一点疑问：vfk的程序里面似乎故意忽略掉了关于高度平衡的优化，并且事实证明这样写更优，不知道是何原因……

关于替罪羊树，参见vfk的`《对无旋转操作的平衡树的一些探究》`
关于线段树的合并，参见主席的`《线段树的合并》`

<!--more-->

## 代码
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
using namespace std;
 
const int MaxN = 1e5;
const int MaxV = 2.3e7;
const int MaxValue = 70001;
 
const double DepthLim = log(4) - log(3);
const double SizeLim = (double)3 / 4;
 
int value[MaxN], n, q, lastans = 0;
 
namespace seg {
    struct node {
        node *l, *r;
        int cnt, size;
    };
 
    node pool[MaxV];
    node *notUsed[MaxV], *null;
    int top, last;
 
    inline void init() {
        last = 1; top = 0;
        null = pool;
        null->l = null->r = null;
        null->cnt = null->size = 0;
    }
 
    inline node* getNew() {
        node *res;
        if (top)
            res = notUsed[top--];
        else {
            if (++last >= MaxV) {
                printf("Wrong! Seg pool is empty!\n");
                return 0;
            } else
                res = pool + last;
        } 
        res->l = res->r = null;
        res->cnt = res->size = 1;
        return res;
    }
 
    void delAll(node *now) {
        if (now == null) return;
        if (!(--now->cnt)) {
            notUsed[++top] = now;
            delAll(now->l);
            delAll(now->r);
        }
    }
 
    node* merge(node *a, node *b) {
        if (a == null) return b->cnt++, b;
        if (b == null) return a->cnt++, a;
        node *res = getNew();
        if (a->l == null && a->r == null) 
            res->size = a->size + b->size;    
        else {
            node *l = merge(a->l, b->l);
            node *r = merge(a->r, b->r);
            res->l = l; res->r = r;
            res->size = l->size + r->size;
        }
        return res;
    }
 
    node* add(node *pre, int l, int r, int x, int v) {
        node *res = getNew();
        res->size = pre->size + v;
        if (l < r) {
            int mid = (l + r) >> 1;
            if (x <= mid) {
                res->l = add(pre->l, l, mid, x, v);
                res->r = pre->r; 
                pre->r->cnt++;
            } else {
                res->r = add(pre->r, mid + 1, r, x, v);
                res->l = pre->l;
                pre->l->cnt++;
            }
            res->size = res->l->size + res->r->size;
        }
        return res;
    }
};
 
namespace scap {
    struct node {
        node *l, *r;
        seg::node *myseg;
        int size, value;
 
        void update() {
            size = l->size + r->size + 1;
            seg::node *tmp = seg::merge(l->myseg, r->myseg);
            myseg = seg::add(tmp, 1, MaxValue, value, 1);
            seg::delAll(tmp);
        }
    };
 
    node pool[MaxN], *null, *tmpnode, *root;
    seg::node *check[MaxN];
    int checkint[MaxN];
    int last, maxDepth, top, topint;
 
    inline void init() {
        last = 1; 
        root = null = pool;
        tmpnode = pool + 1;
        null->l = null->r = null;
        null->myseg = seg::null;
        null->size = 0;
    }
 
    inline node* getNew() {
        node *res = pool + ++last;
        res->l = res->r = null;
        res->myseg = seg::null;
        res->size = 1;
        return res;
    }
 
    node* flatten(node *x, node *y) {
        if (x == null) return y;
        seg::delAll(x->myseg);
        x->r = flatten(x->r, y);
        return flatten(x->l, x);
    }
 
    node* build(node *x, int n) {
        if (!n) {
            x->l = null;
            return x;
        }
        node *y = build(x, n >> 1);
        node *z = build(y->r, n - 1 - (n >> 1));
        y->r = z->l;
        z->l = y;
        y->update();
        return z;
    }
 
    node* rebuild(node *now) {
        int size = now->size;
        node *t = tmpnode;
        now = flatten(now, t);
        return build(now, size)->l;
    }
 
    bool insert2(node *&now, int x, int v, int depth) {
        if (now == null) {
            now = getNew();
            now->value = v;
            now->myseg = seg::add(now->myseg, 1, MaxValue, v, 1);
            return depth > maxDepth;
        }
 
        seg::node *tmp = now->myseg;
        now->myseg = seg::add(tmp, 1, MaxValue, v, 1);
        seg::delAll(tmp);
        now->size++;
 
        bool needRebuilt;
        if (x <= now->l->size + 1)
            needRebuilt = insert2(now->l, x, v, depth + 1);
        else
            needRebuilt = insert2(now->r, x - now->l->size - 1, v, depth + 1);
 
        int st = now->size * SizeLim;
        if (needRebuilt && (now->l->size > st || now->r->size > st)) {
            now = rebuild(now);
            return false;
        } else {
            return needRebuilt;
        }
    }
 
    node* initbuild(int l, int r) {
        node *res = getNew();
        int mid = (l + r) >> 1;
        res->value = value[mid];
        if (l < mid) res->l = initbuild(l, mid - 1);
        if (r > mid) res->r = initbuild(mid + 1, r);
        res->update();
        return res;
    }
 
    void insert(int x, int v) {
        maxDepth = log(root->size) / DepthLim + 1;
        insert2(root, x, v, 0);
    }
 
    int change(node *now, int x, int v) {
        int st = now->l->size, lastv;
        if (x <= st)
            lastv = change(now->l, x, v);
        else if (x > st + 1)
            lastv = change(now->r, x - st - 1, v);
        else
            lastv = now->value, now->value = v;
         
        seg::node *tmp = now->myseg;
        now->myseg = seg::add(tmp, 1, MaxValue, lastv, -1);
        seg::delAll(tmp);
        tmp = now->myseg;
        now->myseg = seg::add(tmp, 1, MaxValue, v, 1);
        seg::delAll(tmp);
         
        return lastv;
    }
 
    void query2(node *now, int l, int r) {
        if (l <= 1 && now->size <= r) {
            check[++top] = now->myseg;
            return;
        }
        int mid = now->l->size + 1;
        if (l <= mid && mid <= r)
            checkint[++topint] = now->value;
        if (l < mid) query2(now->l, l, r);
        if (r > mid) query2(now->r, l - mid, r - mid);
    }
 
    int query(int l, int r, int k) {
        top = topint = 0;
        query2(root, l, r);
 
        l = 1, r = MaxValue;
        while (l < r) {
            int mid = (l + r) >> 1, tmp = 0;
            for (int i = 1; i <= top; i++)
                tmp += check[i]->l->size;
            for (int i = 1; i <= topint; i++)
                if (l <= checkint[i] && checkint[i] <= mid)
                    tmp++;
            if (k <= tmp) {
                for (int i = 1; i <= top; i++)
                    check[i] = check[i]->l;
                r = mid;
            }
            else {
                for (int i = 1; i <= top; i++)
                    check[i] = check[i]->r;
                k -= tmp;
                l = mid + 1;
            }
        }
        return l;
    }
};
 
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
 
inline char getmychar() {
    char c;
    while (!((c = getchar()) >= 'A' && c <= 'Z'));
    return c;
}
 
void mck() {
    int tmp = sizeof(seg::pool) + sizeof(seg::notUsed) + sizeof(scap::pool) 
            + sizeof(scap::check) + sizeof(scap::checkint);
    printf("%d\n", tmp / 1000000);
}
 
void print(scap::node *now) {
    if (now == scap::null) return;
    print(now->l);
    printf("%d ", now->value - 1);
    print(now->r);
}
 
void Print() {
    print(scap::root);
    putchar('\n');
}
 
int main() {
    seg::init();
    scap::init();
 
    n = getnum();
    for (int i = 1; i <= n; i++) value[i] = getnum() + 1;
    scap::root = scap::initbuild(1, n);
    q = getnum();
    for (int i = 1; i <= q; i++) {
        //Print();
        char intype = getmychar();
        int x, y, z;
        switch (intype) {
            case 'Q':
                x = getnum() ^ lastans;
                y = getnum() ^ lastans;
                z = getnum() ^ lastans;
                //printf("Q %d %d %d\n", x, y, z);
                if (x > y) swap(x, y);
                lastans = scap::query(x, y, z) - 1;
                printf("%d\n", lastans);
                break;
            case 'M':
                x = getnum() ^ lastans; 
                y = (getnum() ^ lastans) + 1;
                //printf("M %d %d\n", x, y - 1);
                scap::change(scap::root, x, y);
                break;
            case 'I':
                x = getnum() ^ lastans;
                y = (getnum() ^ lastans) + 1;
                //printf("I %d %d\n", x, y - 1);
                scap::insert(x, y);
                break;
        }
    }
}
```