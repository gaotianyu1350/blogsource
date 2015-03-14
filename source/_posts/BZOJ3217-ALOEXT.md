title: "[Solution][BZOJ3217]ALOEXT"
date: 2015-03-05 16:02:38
tags: [替罪羊树,可持久化Trie,线段树合并,BZOJ]
categories: 题解
---

## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3217)

## 分析

同[BZOJ3065](http://gaotianyu1350.gitcafe.io/2015/03/05/BZOJ3065-%E5%B8%A6%E6%8F%92%E5%85%A5%E5%8C%BA%E9%97%B4k%E5%B0%8F%E5%80%BC/)

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
 
const int MaxN = 2e5 + 100;
const int MaxV = 1048576;
const int MaxS = 2e7 + 100;
 
const double LogAlpha = log(4) - log(3);
const double SizeRate = (double)3 / 4;
 
int value[MaxN], n, m;
 
namespace seg {
    struct node {
        int cnt, size;
        node *l, *r;
    };
 
    node pool[MaxS], *notUsed[MaxS], *poolTail, *null;
    int top;
 
    inline void init() {
        top = 0; null = poolTail = pool;
        null->l = null->r = null;
        null->cnt = null->size = 0;
    }
 
    inline node* getNew() {
        node *res = 0;
        if (top)
            res = notUsed[top--];
        else if (++poolTail < pool + MaxS)
            res = poolTail;
        else {
            printf("Wrong! Seg pool is empty!\n");
            return 0;
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
            int mid = ((l + r + 1) >> 1) - 1;
            if (x <= mid) {
                res->l = add(pre->l, l, mid, x, v);
                res->r = pre->r;
                pre->r->cnt++;
            } else {
                res->r = add(pre->r, mid + 1, r, x, v);
                res->l = pre->l;
                pre->l->cnt++;
            }
        }
        return res;
    }
};
 
namespace scap {
    struct node {
        int size, value;
        node *l, *r;
        seg::node *myseg;
 
        void update() {
            seg::node *tmp = seg::merge(l->myseg, r->myseg);
            myseg = seg::add(tmp, 0, MaxV - 1, value, 1);
            seg::delAll(tmp);
 
            size = l->size + r->size + 1;
        }
    };
 
    node pool[MaxN], *poolTail, *null, *tmpNode, *root;
 
    seg::node *checkTree[MaxN], *checkOri[MaxN];
    int checkInt[MaxN];
    int topTree, topInt;
    int maxDepth, maxSize;
 
    inline void init() {
        root = null = poolTail = pool;
        null->l = null->r = null;
        null->size = null->value = 0;
        null->myseg = seg::null;
 
        tmpNode = ++poolTail;
    }
 
    inline node* getNew() {
        node *res;
        if (++poolTail < pool + MaxN)
            res = poolTail;
        else {
            printf("Wrong! Scap pool is empty!\n");
            return 0;
        }            
        res->l = res->r = null;
        res->size = 1;
        res->myseg = seg::null;
        return res;
    }
 
    //将以x为根的子树拍扁成一条链，在末尾接上y，返回链的第一个元素
    //拍扁过程中清空所有的线段树
    node* flatten(node *x, node *y) {
        if (x == null) return y;
        seg::delAll(x->myseg);
        x->r = flatten(x->r, y);
        return flatten(x->l, x);
    }
 
    //将以x开头，向后n个元素的链构造成一棵平衡的树，返回
    //第n + 1个节点，节点的左儿子是树根
    //不要忘记update()
    node* buildTree(node *x, int n) {
        if (!n) {
            x->l = null;
            return x;
        }
        node *y = buildTree(x, n >> 1);
        node *z = buildTree(y->r, n - 1 - (n >> 1));
        y->r = z->l;
        z->l = y;
 
        y->update();
        return z;
    }
 
    //重建以now为根的子树，不要忘记将now替换为新的树根
    node* rebuild(node *now) {
        int size = now->size;
        now = flatten(now, tmpNode);
        return buildTree(now, size)->l;
    }
 
    node* build(int l, int r) {
        node *res = getNew();
        int mid = (l + r) >> 1;
        res->value = value[mid];
        if (l < mid) res->l = build(l, mid - 1);
        if (mid < r) res->r = build(mid + 1, r);
         
        res->update();
        return res;
    }
 
    bool insert2(node *&now, int x, int v, int depth) {
        if (now == null) {
            now = getNew();
            now->value = v;
            now->myseg = seg::add(seg::null, 0, MaxV - 1, v, 1);
            return depth <= maxDepth;
        }
        now->size++;
        seg::node *tmp = now->myseg;
        now->myseg = seg::add(tmp, 0, MaxV - 1, v, 1);
        seg::delAll(tmp);
 
        bool needRebuilt;
        int st = now->l->size + 1;
        if (x <= st)
            needRebuilt = insert2(now->l, x, v, depth);
        else
            needRebuilt = insert2(now->r, x - st, v, depth);
 
        int maxSize = now->size * SizeRate;
        if (needRebuilt && (now->l->size > maxSize || now->r->size > maxSize)) {
            now = rebuild(now);
            needRebuilt = false;
        }
        return needRebuilt;
    }
 
    void insert(int x, int v) {
        maxDepth = log(maxSize) / LogAlpha + 1;
        insert2(root, x, v, 0);
        maxSize++;
    }
 
    int change(node *now, int x, int v) {
        int lastValue, st = now->l->size;
        if (x <= st)
            lastValue = change(now->l, x, v);
        else if (x > st + 1)
            lastValue = change(now->r, x - st - 1, v);
        else
            lastValue = now->value, now->value = v;
 
        seg::node *tmp = now->myseg;
        now->myseg = seg::add(tmp, 0, MaxV - 1, lastValue, -1);
        delAll(tmp);
        tmp = now->myseg;
        now->myseg = seg::add(tmp, 0, MaxV - 1, v, 1);
        delAll(tmp);
 
        return lastValue;       
    }
 
    int del(node *&now, int x) {
        int lastValue, st = now->l->size;
        if (x <= st)
            lastValue = del(now->l, x);
        else if (x > st + 1)
            lastValue = del(now->r, x - st - 1);
        else {
            lastValue = now->value;
            if (now->l == null && now->r == null) {
                seg::delAll(now->myseg);
                now = null;
                return lastValue;
            }
            else if (now->l == null) {
                seg::delAll(now->myseg);
                now = now->r;
                return lastValue;
            }
            else if (now->r == null) {
                seg::delAll(now->myseg);
                now = now->l;
                return lastValue;
            }
            else {
                int tValue = del(now->r, 1);
                now->value = tValue;
            }
        }
 
        now->size--;
        seg::node *tmp = now->myseg;
        now->myseg = seg::add(tmp, 0, MaxV - 1, lastValue, -1);
        seg::delAll(tmp);
        return lastValue;
    }
 
    void query2(node *now, int l, int r) {
        if (now == null) return;
        if (l <= 1 && r >= now->size) {
            checkOri[++topTree] = now->myseg;
            return;
        }
        int mid = now->l->size + 1;
        if (l <= mid && mid <= r)
            checkInt[++topInt] = now->value;
        if (l < mid) query2(now->l, l, r);
        if (r > mid) query2(now->r, l - mid, r - mid);
    }
 
    int query(int l, int r, int k) {
        topTree = topInt = 0;
        query2(root, l, r);
 
        //Find Kth
        int left = 0, right = MaxV - 1;
        memcpy(checkTree + 1, checkOri + 1, sizeof(seg::node*) * topTree);
        while (left < right) {
            int mid = ((left + right + 1) >> 1) - 1;
            int tmp = 0;
            for (int i = 1; i <= topTree; i++)
                tmp += checkTree[i]->l->size;
            for (int i = 1; i <= topInt; i++)
                tmp += (checkInt[i] <= mid && checkInt[i] >= left ? 1 : 0);
 
            if (k <= tmp) {
                for (int i = 1; i <= topTree; i++)
                    checkTree[i] = checkTree[i]->l;
                right = mid;
            } else {
                for (int i = 1; i <= topTree; i++)
                    checkTree[i] = checkTree[i]->r;
                k -= tmp;
                left = mid + 1;
            }
        }
 
        int key = left, key2 = left;
        left = 0, right = MaxV - 1;
        memcpy(checkTree + 1, checkOri + 1, sizeof(seg::node*) * topTree);
        while (left < right) {
            int mid = ((left + right + 1) >> 1) - 1, tmp = 0, dir;
            if (key > mid) {
                for (int i = 1; i <= topTree; i++)
                    tmp += checkTree[i]->l->size;
                for (int i = 1; i <= topInt; i++)
                    tmp += (checkInt[i] <= mid && checkInt[i] >= left ? 1 : 0);                
                if (tmp)
                    key -= mid - left + 1, dir = 0;
                else
                    dir = 1;
            } else {
                for (int i = 1; i <= topTree; i++)
                    tmp += checkTree[i]->r->size;
                for (int i = 1; i <= topInt; i++)
                    tmp += (checkInt[i] > mid && checkInt[i] <= right ? 1 : 0);
                if (tmp)
                    key += mid - left + 1, dir = 1;
                else
                    dir = 0;
            }
 
            if (!dir) {
                for (int i = 1; i <= topTree; i++)
                    checkTree[i] = checkTree[i]->l;
                right = mid;
            }
            else {
                for (int i = 1; i <= topTree; i++)
                    checkTree[i] = checkTree[i]->r;
                left = mid + 1;
            }
        }
 
        return key2 ^ left;
    }
 
    void print(node *now) {
        if (now == null) return;
        print(now->l);
        printf("%d ", now->value);
        print(now->r);
    }
 
    void Print() {
        print(root);
        putchar('\n');
    }
};
 
inline void mck() {
    int tar = sizeof(value) + sizeof(seg::pool) + sizeof(seg::notUsed)
            + sizeof(scap::pool) + sizeof(scap::checkOri) + sizeof(scap::checkInt)
            + sizeof(scap::checkTree);
    printf("%d\n", tar / 1000000);
}
 
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
 
int main() {
    seg::init();
    scap::init();
 
    n = getnum(); m = getnum();
    for (int i = 1; i <= n; i++)
        value[i] = getnum();
    scap::root = scap::build(1, n);
    scap::maxSize = scap::root->size;
 
    int lastAns = 0;
    for (int t = 1; t <= m; t++) {
        //scap::Print();
        char intype = getmychar();
        int x, y;
        switch (intype) {
            case 'I':
                x = (getnum() + lastAns) % scap::root->size + 1; 
                y = (getnum() + lastAns) % MaxV;
                scap::insert(x, y);
                break;
            case 'D':
                x = (getnum() + lastAns) % scap::root->size + 1;
                scap::del(scap::root, x);
                break;
            case 'C':
                x = (getnum() + lastAns) % scap::root->size + 1; 
                y = (getnum() + lastAns) % MaxV;
                scap::change(scap::root, x, y);
                break;
            case 'F':
                x = (getnum() + lastAns) % scap::root->size + 1; 
                y = (getnum() + lastAns) % scap::root->size + 1; 
                if (x > y) swap(x, y);
                if (x == y)
                    printf("Wrong! Bad data!\n");
                lastAns = scap::query(x, y, (y - x + 1) - 1);
                printf("%d\n", lastAns);
                //lastAns = 0;
                break;
        }
    }
}
```