title: "[Solution][BZOJ3569]DZY Loves Chinese II"
date: 2015-04-02 10:00:07
tags: [BZOJ,生成树,Hash,随机,xor,线性基]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3569)

## 分析
关于动态图的连通性问题……一看到就吓尿了。其实做法很简单也很巧妙。

首先我们随便弄一个生成树出来。如果砍掉的是一条树边，在什么情况下会导致整棵树不连通？只有在没有非树边覆盖它或者所有覆盖它的非树边都被砍掉时会不连通。对于每一条非树边，我们随机一个权值，然后让树边的权值等于所有覆盖它的非树边的权值的异或和。对于每一次询问，我们求一下线性基，然后看一下从中选出几个值，是否可能异或和为$0$。如果可能，就说明会不连通。

**细节**：如果快速求树边的权值。给每个点一个权值，为所有与它相连的非树边的权值。对于每条树边，它的权值为树边上深度较深的那个点的子树的权值和。

**易错**：像这种询问信息一长串，而且有可能不读入完就能得到答案的题，千万不要从中间`break`！因为后面还有数字需要读，如果直接退出会导致输入混乱。

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

const int MaxN = 1e5 + 10;
const int MaxM = 5e5 + 10;
const int MaxBit = 31;

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

inline int getrand() {
    return rand() * rand();
}

struct edge {
    int x, y, value;
    inline void get() {
        x = getnum(); y = getnum();
    }
} e[MaxM];

int n, m, q, lastAns;
int point[MaxN], v[MaxN * 2], nxt[MaxN * 2], idx[MaxN * 2], tot;
int delta[MaxN];

inline void addedge(int x, int y, int id) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y; idx[tot] = id;
    tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x; idx[tot] = id;
}

namespace un {
    int father[MaxN];

    inline void init() {
        for (int i = 1; i <= n; i++)
            father[i] = i;
    }

    int getfather(int x) {
        if (father[x] == x) return x;
        return father[x] = getfather(father[x]);
    }
}

namespace lbase {
    int b[MaxBit];

    inline void init() {
        memset(b, 0, sizeof(b));
    }

    inline bool add(int x) {
        for (int i = MaxBit - 1; i >= 0; i--)
            if (x >> i & 1) {
                if (!b[i]) {
                    b[i] = x;
                    return false;
                } else {
                    x ^= b[i];
                    if (!x) return true;
                }
            }

        return false;
    }
}

void dfs(int now, int fa) {
    for (int tmp = point[now], u; u = v[tmp], tmp; tmp = nxt[tmp])
        if (u != fa) {
            dfs(u, now);
            e[idx[tmp]].value = delta[u];
            delta[now] ^= delta[u];
        }
}

int main() {
    srand(2333);

    n = getnum(); m = getnum();
    un::init();
    int cnt = 0;
    for (int i = 1; i <= m; i++) {
        e[i].get();
        int fx = un::getfather(e[i].x);
        int fy = un::getfather(e[i].y);

        if (fx == fy) {
            while (!(e[i].value = getrand()));
        } else {
            un::father[fx] = fy;
            addedge(e[i].x, e[i].y, i);
            cnt++;
        }
    }

    for (int i = 1; i <= m; i++)
        if (e[i].value) {
            delta[e[i].x] ^= e[i].value;
            delta[e[i].y] ^= e[i].value;
        }
    dfs(1, 0);

    q = getnum();
    while (q--) {
        int k = getnum();
        lbase::init();

        bool isok = true;
        for (int i = 1; i <= k; i++) {
            int idx = getnum() ^ lastAns;
            if (isok && (!e[idx].value || lbase::add(e[idx].value)))
                isok = false;
        }

        if (cnt != n - 1)
            printf("Disconnected\n");
        else if (isok) {
            printf("Connected\n");
            lastAns++;
        }
        else
            printf("Disconnected\n");
    }
}
```