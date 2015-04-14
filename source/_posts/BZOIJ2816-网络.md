title: "[Solution][BZOJ2816][ZJOI2012]网络"
date: 2015-04-02 18:33:36
tags: [BZOJ,ZJOI,Splay]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=2816)

## 分析
这题`BZOJ`上没有题面，想看题面可以去[这里](http://115.com/file/be1pwrfz)

注意题目中的描述：每个点同种颜色的边最多只会有两个。这就意味着同种颜色的边构成的是一些链。然后我们要支持合并链，断开链，链上求最大的操作，经典的`splay`嘛。

有两个需要**注意**的地方：①合并的时候，两个链的方向可能不同，所以需要把它们转成方向合适，维护一个$rev$标记。②询问的时候可能有$u=v$的情况。需要特判一下。

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

const int MaxN = 1e4 + 10;
const int MaxM = 1e5 + 10;
const int MaxC = 12;

const int MaxNode = MaxN * 10;

int n, m, cntColor, k, value[MaxN];

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

namespace output {
    inline void Success() {
        printf("Success.\n");
    }

    inline void Error1() {
        printf("Error 1.\n");
    }

    inline void Error2() {
        printf("Error 2.\n");
    }

    inline void NoEdge() {
        printf("No such edge.\n");
    }
}

namespace gra {
    vector<int> edge[MaxN][MaxC];

    inline void addEdge(int x, int y, int c) {
        edge[x][c].push_back(y);
        edge[y][c].push_back(x);
    }

    inline int hasEdge(int x, int y) {
        for (int c = 1; c <= cntColor; c++) {
            vector<int> &cur = edge[x][c];
            for (int i = 0; i < (int)cur.size(); i++)
                if (cur[i] == y)
                    return c;
        }
        return 0;
    }

    inline int nxt(int x, int c, int avoid) {
        vector<int> &cur = edge[x][c];
        for (int i = 0; i < (int)cur.size(); i++)
            if (cur[i] != avoid)
                return cur[i];
        return -1;
    }

    inline int cnt(int x, int c) {
        return edge[x][c].size();
    }

    inline void delEdge(int x, int y, int c) {
        vector<int> &curx = edge[x][c];
        for (int i = 0; i < (int)curx.size(); i++)
            if (curx[i] == y) {
                curx.erase(curx.begin() + i);
                break;
            }

        vector<int> &cury = edge[y][c];
        for (int i = 0; i < (int)cury.size(); i++)
            if (cury[i] == x) {
                cury.erase(cury.begin() + i);
                break;
            }
    }
}

namespace splay {
    struct splay_node {
        int idx, value, rev, maxx;
        splay_node *pre, *ch[2];

        void pushdown() {
            if (rev) {
                swap(ch[0], ch[1]);
                ch[0]->rev ^= 1;
                ch[1]->rev ^= 1;
                rev = 0;
            }
        }

        void update() {
            maxx = max(value, max(ch[0]->maxx, ch[1]->maxx));
        }

        void setch(int wh, splay_node *child) {
            pushdown();
            child->pre = this;
            ch[wh] = child;
            update();
        }

        int getwh() {
            return pre->ch[0] == this ? 0 : 1;
        }

        int getempty();
    } pool[MaxNode], *null;

    int splay_node::getempty() {
        if (ch[0] == null) return 0;
        if (ch[1] == null) return 1;
        return -1;
    }

    inline splay_node *getNode(int x, int c) {
        return pool + (c - 1) * n + x;
    }

    inline void init() {
        null = pool;
        null->idx = 0;
        null->value = null->rev = null->maxx = 0;
        null->pre = null->ch[0] = null->ch[1] = null;

        int cntNode = n * cntColor;
        for (int i = 1, idx = 1; i <= cntNode; i++, idx++) {
            splay_node *cur = pool + i;
            if (idx > n) idx = 1;

            cur->idx = idx;
            cur->value = cur->maxx = value[idx];
            cur->rev = 0;
            cur->pre = cur->ch[0] = cur->ch[1] = null;
        }
    }

    inline void rotate(splay_node *now) {
        splay_node *oldf = now->pre, *grand = now->pre->pre;
        grand->pushdown(); oldf->pushdown(); now->pushdown();
        int wh = now->getwh();

        oldf->setch(wh, now->ch[wh ^ 1]);
        now->setch(wh ^ 1, oldf);
        now->pre = grand;
        if (grand != null)
            grand->ch[grand->ch[0] == oldf ? 0 : 1] = now;
        grand->update();
    }

    inline void splay(splay_node *now, splay_node *tar) {
        for (; now->pre != tar && now->pre != null; rotate(now))
            if (now->pre->pre != tar && now->pre->pre != null)
                now->getwh() == now->pre->getwh() ? rotate(now->pre) : rotate(now);
    }

    splay_node* build(int *tmp, int l, int r, int c) {
        if (l > r) return null;

        int mid = (l + r) >> 1;
        splay_node *now = getNode(tmp[mid], c);
        now->setch(0, build(tmp, l, mid - 1, c));
        now->setch(1, build(tmp, mid + 1, r, c));

        return now;
    }

    inline void build_color(int c) {
        static bool check[MaxN];
        static int tmp[MaxN], top;
        
        memset(check, 0, sizeof(check));
        
        for (int i = 1; i <= n; i++)
            if (!check[i] && gra::cnt(i, c) == 1) {
                top = 0;
                int now = i, last = -1;
                while (now != -1) {
                    tmp[++top] = now;
                    check[now] = true;

                    int nxt = gra::nxt(now, c, last);
                    last = now;
                    now = nxt;
                }

                build(tmp, 1, top, c);
            }
    }

    inline void changeValue(int x, int v) {
        for (int c = 1; c <= cntColor; c++) {
            splay_node *now = getNode(x, c);
            now->value = v;
            while (now != null) {
                now->update();
                now = now->pre;
            }
        }
    }
}

int main() {
    // Input
    n = getnum(); m = getnum(); cntColor = getnum(); k = getnum();
    for (int i = 1; i <= n; i++) value[i] = getnum();
    for (int i = 1; i <= m; i++) {
        int x = getnum();
        int y = getnum();
        int w = getnum() + 1;
        gra::addEdge(x, y, w);
    }

    // Init
    splay::init();
    for (int i = 1; i <= cntColor; i++)
        splay::build_color(i);

    // Solve Queries
    while (k--) {
        int intype = getnum();

        if (intype == 0) {
            int x = getnum(); int y = getnum();
            splay::changeValue(x, y);
            value[x] = y;
        } else if (intype == 1) {
            int u = getnum(); int v = getnum(); int c = getnum() + 1;
            int oldEdgeColor = gra::hasEdge(u, v);

            // <A>
            if (!oldEdgeColor) {
                output::NoEdge();
                continue;
            }

            // Check If the Old Color Is the Same as the New Color
            if (c == oldEdgeColor) {
                output::Success();
                continue;
            }

            // <B>
            if (gra::cnt(u, c) > 1 || gra::cnt(v, c) > 1) {
                output::Error1();
                continue;
            }

            // <C>
            splay::splay_node *unode = splay::getNode(u, c);
            splay::splay_node *vnode = splay::getNode(v, c);
            
            splay::splay(unode, splay::null);
            splay::splay(vnode, splay::null);
            
            if (unode->pre != splay::null) {
                output::Error2();
                continue;
            }

            // <D>
            output::Success();

            // -- First : Reverse and Link --
            unode->pushdown();
            vnode->pushdown();
            if (unode->getempty() == vnode->getempty())
                vnode->rev ^= 1;
            unode->setch(unode->getempty(), vnode);
            gra::addEdge(u, v, c);

            // -- Second : Delete the Old Edge --
            splay::splay_node *oldu = splay::getNode(u, oldEdgeColor);
            splay::splay_node *oldv = splay::getNode(v, oldEdgeColor);
            splay::splay(oldu, splay::null);
            splay::splay(oldv, oldu);
            oldu->pushdown();
            oldv->pushdown();

            int wh = oldv->getwh();
            oldu->setch(wh, splay::null);
            oldv->pre = splay::null;
            gra::delEdge(u, v, oldEdgeColor);
        } else {
            int c = getnum() + 1; int u = getnum(); int v = getnum();
            // Check If u == v
            if (u == v) {
                printf("%d\n", value[u]);
                continue;
            }

            splay::splay_node *unode = splay::getNode(u, c);
            splay::splay_node *vnode = splay::getNode(v, c);
            splay::splay(unode, splay::null);
            splay::splay(vnode, unode);

            if (vnode->pre == splay::null) {
                printf("-1\n");
            } else {
                unode->pushdown();
                vnode->pushdown();
                int ans = vnode->ch[vnode->getwh() ^ 1]->maxx;
                ans = max(ans, value[u]);
                ans = max(ans, value[v]);
                printf("%d\n", ans);
            }
        }
    }
}

```