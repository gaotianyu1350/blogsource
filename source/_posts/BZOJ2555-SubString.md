title: "[Solution][BZOJ2555]SubString"
date: 2015-04-08 19:18:02
tags: [BZOJ,SAM,LCT]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=2555)

## 分析
如果不看修改操作，明显的`SAM`。然后有一个向后加的操作。由于`SAM`可以直接往后加，所以没有问题。蛋疼的是出现次数是没法直接动态维护的，如果新加入一个末尾节点，就需要将从它到根的所有点的出现次数$+1$，还要考虑换父亲的操作，所以用`LCT`维护一下`Parent`树（也可以用`Splay`+`DFS序`）。

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

const int MaxLen = 6e5 + 10;
const int MaxNode = 1.2e6 + 10;

int Right[MaxNode];

namespace lct {
    struct lct_node {
        lct_node *pre, *ch[2];
        int idx, rev, delta;
        void pushdown();
        void setch(int, lct_node*);
        bool isroot();
        int getwh();
    } *null, pool[MaxNode];

    void lct_node::pushdown() {
        if (rev) {
            swap(ch[0], ch[1]);
            if (ch[0] != null) ch[0]->rev ^= 1;
            if (ch[1] != null) ch[1]->rev ^= 1;
            rev = 0;
        }
        if (delta) {
            if (ch[0] != null) Right[ch[0]->idx] += delta, ch[0]->delta += delta;
            if (ch[1] != null) Right[ch[1]->idx] += delta, ch[1]->delta += delta;
            delta = 0;
        }
    }

    void lct_node::setch(int wh, lct_node *child) {
        pushdown();
        ch[wh] = child;
        if (child != null)
            child->pre = this;
    }

    bool lct_node::isroot() {
        return pre->ch[0] != this && pre->ch[1] != this;
    }

    int lct_node::getwh() {
        return pre->ch[0] == this ? 0 : 1;
    }

    inline void init() {
        null = new lct_node();
        null->pre = null->ch[0] = null->ch[1] = null;
        null->idx = null->rev = null->delta = 0;

    }

    inline void initnew(int idx) {
        lct_node *res = pool + idx;
        res->pre = res->ch[0] = res->ch[1] = null;
        res->idx = idx; res->rev = res->delta = 0;
    }

    inline void rotate(lct_node *now) {
        lct_node *oldf = now->pre, *grand = now->pre->pre;
        grand->pushdown(); oldf->pushdown(); now->pushdown();
        int wh = now->getwh(); bool isr = oldf->isroot();
        
        oldf->setch(wh, now->ch[wh ^ 1]);
        now->setch(wh ^ 1, oldf);
        now->pre = grand;

        if (!isr) grand->setch(grand->ch[0] == oldf ? 0 : 1, now);
    }

    inline void splay(lct_node *now) {
        for (; !now->isroot(); rotate(now))
            if (!now->pre->isroot())
                now->getwh() == now->pre->getwh() ? rotate(now->pre) : rotate(now);
    }

    inline lct_node *access(lct_node *now) {
        lct_node *last = null;
        for (; now != null; last = now, now = now->pre) {
            splay(now);
            now->setch(1, last);
        }
        return last;
    }

    inline void changeroot(lct_node *now) {
        access(now)->rev ^= 1;
        splay(now);
    }

    inline void link(lct_node *x, lct_node *y) {
        changeroot(y);
        y->pre = x;
        access(y);
    }

    inline void cut(lct_node *x, lct_node *y) {
        changeroot(x);
        access(y);
        splay(y);
        y->setch(0, null);
        x->pre = null;
    }

    inline void add(lct_node *root, lct_node *x, int v) {
        changeroot(root);
        access(x);
        splay(x);
        Right[x->idx] += v;
        x->delta += v;
    }

    inline int query(lct_node *x) {
        splay(x);
        return Right[x->idx];
    }

    inline lct_node *getnode(int x) {
        return pool + x;
    }
}

namespace sam {
    struct sam_node {
        int val, idx;
        sam_node *pre, *ch[26];
        sam_node() {
            memset(ch, 0, sizeof(ch));
            pre = 0; 
            val = idx = 0;
        }
    } *last, *root, pool[MaxNode];
    int cnt = 0;

    inline sam_node *getnew() {
        sam_node *res = pool + (cnt++);
        lct::initnew(cnt - 1);
        res->idx = cnt - 1;
        return res;
    }

    inline void init() {
        last = root = pool;
        getnew();
    }

    inline void link(sam_node *x, sam_node *y) {
        lct::link(lct::getnode(x->idx), lct::getnode(y->idx));
    }

    inline void cut(sam_node *x, sam_node *y) {
        lct::cut(lct::getnode(x->idx), lct::getnode(y->idx));
    }

    inline void add(int c) {
        sam_node *p = last;
        sam_node *np = getnew();
        np->val = p->val + 1;
        while (p && !p->ch[c])
            p->ch[c] = np, p = p->pre;
        if (!p) {
            np->pre = root;
            link(np, root);
        }
        else {
            sam_node *q = p->ch[c];
            if (q->val == p->val + 1) {
                np->pre = q;
                link(np, q);
            }
            else {
                sam_node *nq = getnew();
                nq->val = p->val + 1;
                Right[nq->idx] = lct::query(lct::getnode(q->idx));
                memcpy(nq->ch, q->ch, sizeof(q->ch));

                link(nq, q->pre);
                cut(q, q->pre);
                link(q, nq);
                link(np, nq);

                nq->pre = q->pre;
                q->pre = np->pre = nq;

                while (p && p->ch[c] == q)
                    p->ch[c] = nq, p = p->pre;
            }
        }
        last = np;
        lct::add(lct::getnode(root->idx), lct::getnode(np->idx), 1);
    }

    inline int query(const char *s, int len) {
        sam_node *now = root;
        bool isok = true;
        for (int i = 0; i < len; i++) {
            int cur = s[i] - 'A';
            if (now->ch[cur])
                now = now->ch[cur];
            else {
                isok = false;
                break;
            }
        }
        if (!isok) return 0;
        else return lct::query(lct::getnode(now->idx));
    }
}

int mask;

void decode(char *s, int len, int mask) {
    for (int i = 0; i < len; i++) {
        mask = (mask * 131 + i) % len;
        swap(s[i], s[mask]);
    }
}

void getstring(char *s, int &len) {
    len = 0;
    while (1) {
        char c = getchar();
        if (!len)
            while (!isupper(c)) c = getchar();
        if (isupper(c)) s[len++] = c;
        else break;
    }
    s[len] = '\0';
}

char tmp_s[MaxLen];
int tmp_len;

int main() {
    lct::init();
    sam::init();

    int q;
    scanf("%d", &q);

    getstring(tmp_s, tmp_len);
    for (int i = 0; i < tmp_len; i++)
        sam::add(tmp_s[i] - 'A');

    while (q--) {
        getstring(tmp_s, tmp_len);
        if (!strcmp(tmp_s, "QUERY")) {
            getstring(tmp_s, tmp_len);
            decode(tmp_s, tmp_len, mask);

            int ans = sam::query(tmp_s, tmp_len);
            printf("%d\n", ans);
            mask ^= ans;
        } else {
            getstring(tmp_s, tmp_len);
            decode(tmp_s, tmp_len, mask);

            for (int i = 0; i < tmp_len; i++)
                sam::add(tmp_s[i] - 'A');
        }
    }
}
```