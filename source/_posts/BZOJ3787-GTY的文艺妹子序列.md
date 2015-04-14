title: "[Solution][BZOJ3787]GTY的文艺妹子序列"
date: 2015-03-27 11:01:43
tags: [BZOJ,树状数组,分块,逆序对]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3787)

## 分析
带点修改的区间逆序对查询。

大思路是分块。查询的时候可以将询问区间分成三部分——完整的块，和块两侧的零散的点。对于完整的块，我们可以预处理`f[i][j]`表示块$i$到块$j$之间的逆序对个数来回答询问，`t[i]`维护从开始到块$i$的树状数组，用来处理零散的部分。

但现在有修改。维护`t[i]`是比较简单的，但是对于一个修改，会有$O(n)$级别的`f`发生变动，暴力修改显然不行。我们先来看修改带来的影响。假如说修改$x$点，会有一个$x$与它前面点之间逆序对数个数的变动，还有一个与后面的点的逆序对个数的变动。现在再看对`f`的影响。如果一系列区间的左端点相同，那么“对前面点的变动”是相同的。如果右端点相同，则“对后面点的变动”是相同的。我们可以对每个左端点，每个右端点维护一个树状数组，区间修改点查询，解决！

写对拍的时候手残了，把模$n$写成了模$2$，真是没治了……

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

const int MaxN = 5e4 + 10;
const int MaxSize = 400;
//const int MaxBlock = 100;
const int MaxBlock = MaxN / MaxSize + 20;

namespace tree {
    int start[MaxBlock][MaxBlock], end[MaxBlock][MaxBlock];
    int vt[MaxBlock][MaxN], tmp[MaxN];

    inline void add(int *t, int x, int v, int tot) {
        for (int i = x; i <= tot; i += i & (-i))
            t[i] += v;
    }

    inline int get(int *t, int x) {
        int ans = 0;
        for (int i = x; i > 0; i -= i & (-i))
            ans += t[i];
        return ans;
    }
}

int value[MaxN], f[MaxBlock][MaxBlock];
int bSt[MaxBlock], bEnd[MaxBlock], idx[MaxN];
int n, m, totBlock;

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

inline void init() {
    for (int i = 1; i <= n; i += MaxSize) {
        ++totBlock;
        bSt[totBlock] = i;
        bEnd[totBlock] = min(n, i + MaxSize - 1);
        for (int j = bSt[totBlock]; j <= bEnd[totBlock]; j++)
            idx[j] = totBlock;
    }

    for (int i = 1, cur = 1; i <= n; i++) {
        if (i > bEnd[cur]) {
            cur++;
            memcpy(tree::vt[cur], tree::vt[cur - 1], sizeof(tree::vt[cur - 1]));
        }
        tree::add(tree::vt[cur], value[i], 1, n);
    }

    for (int i = 1; i <= totBlock; i++) {
        memset(tree::tmp, 0, sizeof(tree::tmp));
        for (int j = bSt[i], cur = i; j <= n; j++) {
            if (j > bEnd[cur]) {
                cur++;
                f[i][cur] = f[i][cur - 1];
            }
            f[i][cur] += j - bSt[i] - tree::get(tree::tmp, value[j]);
            tree::add(tree::tmp, value[j], 1, n);
        }
    }

    memset(tree::tmp, 0, sizeof(tree::tmp));
}

int main() {
    n = getnum();
    for (int i = 1; i <= n; i++) value[i] = getnum();
    init();

    m = getnum();
    int lastAns = 0;
    while (m--) {
        int intype, x, y;
        intype = getnum(); 
        x = getnum() ^ lastAns; 
        y = getnum() ^ lastAns;

        if (intype == 0) {
            int lb = idx[x];
            int rb = idx[y];
            if (x != bSt[lb]) lb++;
            if (y != bEnd[rb]) rb--;
            lastAns = 0;

            if (lb <= rb) {
                lastAns += f[lb][rb];
                lastAns += tree::get(tree::start[lb], rb);
                lastAns += tree::get(tree::end[rb], lb);

                int tmptot = 0;
                for (int i = x; i < bSt[lb]; i++) {
                    lastAns += tree::get(tree::vt[rb], value[i] - 1) - tree::get(tree::vt[lb - 1], value[i] - 1);
                    lastAns += tmptot - tree::get(tree::tmp, value[i]);
                    tree::add(tree::tmp, value[i], 1, n);
                    tmptot++;
                }
                for (int i = bEnd[rb] + 1; i <= y; i++) {
                    lastAns += bEnd[rb] - bSt[lb] + 1 - (tree::get(tree::vt[rb], value[i]) - tree::get(tree::vt[lb - 1], value[i]));
                    lastAns += tmptot - tree::get(tree::tmp, value[i]);
                    tree::add(tree::tmp, value[i], 1, n);
                    tmptot++;                    
                }

                for (int i = x; i < bSt[lb]; i++) 
                    tree::add(tree::tmp, value[i], -1, n);
                for (int i = bEnd[rb] + 1; i <= y; i++) 
                    tree::add(tree::tmp, value[i], -1, n);            
            } else {
                for (int i = x; i <= y; i++) {
                    lastAns += i - x - tree::get(tree::tmp, value[i]);
                    tree::add(tree::tmp, value[i], 1, n);
                }
                for (int i = x; i <= y; i++)
                    tree::add(tree::tmp, value[i], -1, n);
            }

            printf("%d\n", lastAns);
            //lastAns = 0;
        } else {
            int cur = idx[x];
            
            for (int i = bSt[cur]; i < x; i++)
                tree::add(tree::tmp, value[i], 1, n);
            int front_delta = tree::get(tree::tmp, value[x]) 
                            - tree::get(tree::tmp, y);
            for (int i = bSt[cur]; i < x; i++)
                tree::add(tree::tmp, value[i], -1, n);

            for (int i = x + 1; i <= bEnd[cur]; i++)
                tree::add(tree::tmp, value[i], 1, n);
            int behind_delta = tree::get(tree::tmp, y - 1)
                             - tree::get(tree::tmp, value[x] - 1);
            for (int i = x + 1; i <= bEnd[cur]; i++)
                tree::add(tree::tmp, value[i], -1, n);

            for (int i = 1; i < cur; i++) {
                int extra = (tree::get(tree::vt[cur - 1], value[x]) - tree::get(tree::vt[i - 1], value[x]))
                          - (tree::get(tree::vt[cur - 1], y) - tree::get(tree::vt[i - 1], y));
                tree::add(tree::start[i], cur, extra + front_delta, totBlock);
            }
            tree::add(tree::start[cur], cur, front_delta, totBlock);

            for (int i = cur + 1; i <= totBlock; i++) {
                int extra = (tree::get(tree::vt[i], y - 1) - tree::get(tree::vt[cur], y - 1))
                          - (tree::get(tree::vt[i], value[x] - 1) - tree::get(tree::vt[cur], value[x] - 1));
                tree::add(tree::end[i], 1, extra + behind_delta, totBlock);
                tree::add(tree::end[i], cur + 1, -extra - behind_delta, totBlock);
            }
            tree::add(tree::end[cur], 1, behind_delta, totBlock);

            for (int i = cur; i <= totBlock; i++) {
                tree::add(tree::vt[i], value[x], -1, n);
                tree::add(tree::vt[i], y, 1, n);
            }

            value[x] = y;
        }
    }
}
```
