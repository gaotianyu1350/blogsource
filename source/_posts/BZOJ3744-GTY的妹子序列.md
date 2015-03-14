title: "[Solution][BZOJ3744]GTY的妹子序列"
date: 2014-11-22 07:55:44
tags: [分块,逆序对,树状数组,BZOJ]
categories: 题解
---
做一道挂着自己名字的题总感觉怪怪的……
<!--more-->
##题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3744)
题目大意：给定长度为$n$的序列，$m$个询问，对每个询问$L_i,R_i$，输出$[L_i,R_i]$这个区间里面的逆序对的数量。
##分析
首先离散化
这种一般数据结构搞不了的乱搞题，直接上分块。`Ans[i][j]`表示第`i`块到第`j`块的逆序对数量（$O(n\sqrt nlogn)$预处理）。处理询问的时候完整的块$O(1)$回答。那其它的部分呢？
注意当多个部分合并的时候，它们的逆序对并不是简单的相加，而是互相之间又有新的逆序对产生，如下图：
![bzoj3744](/img/bzoj3744.png)

这三部分除去本身内部的逆序对，还有`1-2`、`1-3`和`2-3`。`2-3`和`2`、`3`的内部我们可以暴力求解（$O(\sqrt nlogn)$），`1-2`、`1-3`需要我们预处理一个新的数组`nxt[i][j]`表示从第`i`块开始往后比`j`大（或者比`j`小，这取决于你是直接算逆序对还是算正序对然后用总数减去），`pre[i][j]`含义相同。这样只需要$O(\sqrt n)$扫一遍就可以完成了。

要注意一下相等的情况是不算逆序对的。

大脑洞神题……本来是想要用可持久化树状数组，结果`wangxz`神犇的算法要神好多，就用了这个简单的方法……再此YM一下@[wangxz](http://bakser.gitcafe.com/)

##代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
using namespace std;
typedef long long ll;
const int MAXN  = 5 * 1e4 + 10;
const int BLOCK = 400;
const int ARRAYBLOCK = 150;
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
struct num {
    int value, loc, r;
    bool operator < (const num other) const {
        return value < other.value;
    }
}a[MAXN];
ll ans[ARRAYBLOCK][ARRAYBLOCK] = {{0}};
ll prebig[ARRAYBLOCK][MAXN] = {{0}}, nxtsmall[ARRAYBLOCK][MAXN] = {{0}};
ll tree[MAXN] = {0}, lastans = 0;
int n, m;
bool cmp(const num &a, const num &b) {
    return a.loc < b.loc;
}
inline void add(int x, int v) {
    for (int i = x; i <= n; i += (i & (-i))) tree[i] += v;
}
inline ll sum(int x) {
    ll ans = 0;
    for (int i = x; i > 0; i -= (i & (-i))) ans += tree[i];
    return ans;
}
inline int getblock(int x) { return x / BLOCK + (x % BLOCK != 0 ? 1 : 0); }
int main() {
    n = getnum();
    for (int i = 1; i <= n; i++) a[i].value = getnum(), a[i].loc = i;
    sort(a + 1, a + 1 + n);
    a[1].r = 1;
    for (int i = 2; i <= n; i++) a[i].r = a[i].value == a[i - 1].value ? a[i - 1].r : a[i - 1].r + 1;
    sort(a + 1, a + 1 + n, cmp);
    for (int i = 1; i <= n; i += BLOCK) {
        memset(tree, 0, sizeof(tree));
        int last = i, sblock = getblock(i), nowblock = getblock(i);
        for (int j = i; j <= n; j++) {
            if (j >= last + BLOCK) {
                last = j; nowblock++; ans[sblock][nowblock] = ans[sblock][nowblock - 1];
            }
            ans[sblock][nowblock] += sum(a[j].r);
            add(a[j].r, 1);
        }
        for (int j = 1; j <= n; j++) nxtsmall[sblock][j] = sum(j);
        memset(tree, 0, sizeof(tree));
        int tar = min(n, i + BLOCK - 1);
        for (int j = 1; j <= tar; j++) add(a[j].r, 1);
        for (int j = 1; j <= n; j++) prebig[sblock][j] = tar - sum(j - 1);
        //nxtsmall : not strictly smaller
        //prebig   : strictly greater
    }
    m = getnum();
    memset(tree, 0, sizeof(tree));
    for (int t = 1; t <= m; t++) {
        int l, r;
        l = getnum() ^ lastans; r = getnum() ^ lastans;
        lastans = 0;
        if (l > r) swap(l, r);
        int lb = getblock(l), rb = getblock(r);
        int sb, eb;
        if (l == BLOCK * (lb - 1) + 1 && min(l + BLOCK - 1, n) <= r) sb = lb;
        else if (min((lb + 1) * BLOCK, n) <= r) sb = lb + 1;
        else sb = -1;
        if (r == BLOCK * rb && max(1, r - BLOCK + 1) >= l) eb = rb;
        else if (max((rb - 2) * BLOCK + 1, 1) >= l) eb = rb - 1;
        else eb = -1;
        if (sb != -1 && eb != -1) {
            lastans += ans[sb][eb];
            for (int j = l; j <= BLOCK * (sb - 1); j++) {
                lastans += prebig[eb][a[j].r] - prebig[sb - 1][a[j].r];
                lastans += sum(a[j].r); add(a[j].r, 1);
            }
            for (int j = min(eb * BLOCK, n) + 1; j <= r; j++) {
                lastans += nxtsmall[sb][a[j].r] - nxtsmall[eb + 1][a[j].r];
                lastans += sum(a[j].r); add(a[j].r, 1);
            }
            for (int j = l; j <= BLOCK * (sb - 1); j++) add(a[j].r, -1);
            for (int j = min(eb * BLOCK, n) + 1; j <= r; j++) add(a[j].r, -1);
        } else {
            for (int j = l; j <= r; j++) {
                lastans += sum(a[j].r);
                add(a[j].r, 1);
            }
            for (int j = l; j <= r; j++) add(a[j].r, -1);
        }
        lastans = (ll)(r - l + 1) * (r - l) / 2 - lastans;
        printf("%lld\n", lastans);
    }
}
```