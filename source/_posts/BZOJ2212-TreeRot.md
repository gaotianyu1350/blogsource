title: "[Solution][BZOJ2212][POI2011]Tree Rotations"
date: 2015-02-25 09:28:39
tags: [BZOJ,POI,线段树,线段树合并]
categories: 题解
---
<!--more-->
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=2212)

## 分析
对于每一棵子树，我们可以将它对于逆序对数的贡献分为两部分——内部的逆序对和子树与子树外的逆序对。显然，子树与外部逆序对个数与子树内部的构成没有关系。我们只要每次递归的时候考虑左右两颗子树之间的逆序对数即可。

显然平衡树是一种可用的数据结构，递归时启发式合并两棵子树的平衡树，合并的过程中查找每一个平衡树上的节点在另一棵平衡树上的位置即可计算逆序对数，但这样写起来麻烦，而且较慢。

由于点的权值在$1$~$n$的范围内，所以可以用动态开节点的线段树。但如何合并线段树呢？用主席的方法可以轻松做到。大体过程如下：
```
merge(a, b)
	如果a为空节点 返回b
	如果b为空节点 返回a
	如果a,b为叶子节点 返回merge_leaf(a,b)
	返回以merge(a->l, b->l) merge(a->r, b->r)为左右儿子的节点
```
复杂度为两棵线段树共有的节点的数量。对于单次`merge`操作来说，可能不好计算它的时间。但是如果有$n$棵单节点线段树，通过$n-1$次`merge`将它们合并成一棵线段树的时间复杂度为$O(nlogn)$。

对于这个题，我们可以在合并的过程中顺便计算一下交换和不交换的逆序对数。令$ans0$为不交换两棵子树之间的逆序对数，$ans1$为交换后的数量。我们只需要在`merge`的最后一段之前加一句：
```
ans0 = a->r->size * b->l->size
ans1 = a->l->size * b->r->size
```
然后再选取其中较小的一个加入答案即可。

## 代码
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <queue>
#include <algorithm>
using namespace std;

typedef long long ll;
const int MaxN = 4 * 2e5 + 10;
const int MaxNode = 6 * 2e5 + 10;
struct segNode {
    int size, l, r;
    segNode() {
        size = l = r = 0;
    }
    bool isLeaf() {
        return !l && !r;
    }
}seg[MaxNode];
queue<int> pool;
int n, fa[MaxN], ch[MaxN][2], value[MaxN], st[MaxN], top = 0;
int size[MaxN], cur[MaxN], root[MaxN], tot = 0;
inline int newNode() {
    if (pool.empty()) 
        printf("Wrong! There is no more node.\n");
    else {
        int x = pool.front();
        pool.pop();
        return x;
    }
    return 0;
}
inline void deleteNode(int x) {
    seg[x].size = seg[x].l = seg[x].r = 0;
    pool.push(x);
}
inline void init() {
    for (int i = 1; i < MaxNode; i++)
        pool.push(i);
}
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while (((c = getchar()) == ' ' || c == '\n' || c == '\r') && c != -1);
    if (c == -1) return -1;
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
int mergeSeg(int a, int b, ll &ans0, ll &ans1) {
    if (!a) return b;
    if (!b) return a;
    else if (seg[a].isLeaf()) {
        printf("Wrong! Same value\n");
        return 0;
    }
    else {
        ans0 += (ll)seg[seg[a].r].size * seg[seg[b].l].size;
        ans1 += (ll)seg[seg[a].l].size * seg[seg[b].r].size;
        int l = mergeSeg(seg[a].l, seg[b].l, ans0, ans1);
        int r = mergeSeg(seg[a].r, seg[b].r, ans0, ans1);
        int ans = newNode();
        seg[ans].size = seg[l].size + seg[r].size;
        seg[ans].l = l; seg[ans].r = r;
        deleteNode(a); deleteNode(b);
        return ans;
    }
}
inline void dfs() {
    int now = 1;
    memset(cur, -1, sizeof(cur));
    while (now) {
        if (cur[now] >= 0)
            size[now] += size[ch[now][cur[now]]];
        else
            size[now] = 1;
        cur[now]++;
        if (cur[now] == 2 || value[now]) {
            now = fa[now];
            continue;
        }
        now = ch[now][cur[now]];
    }
}
int addtoSeg(int l, int r, int x) {
    int idx = newNode();
    segNode &now = seg[idx];
    if (l == r)
        now.size = 1;
    else {
        int mid = (l + r) >> 1;
        if (x <= mid) now.l = addtoSeg(l, mid, x);
        else now.r = addtoSeg(mid + 1, r, x);
        now.size = 1;
    }
    return idx;
}
inline ll solve() {
    int now = 1;
    ll ans = 0;
    memset(cur, -1, sizeof(cur));
    while (now) {
        if (value[now]) {
            root[now] = addtoSeg(1, n, value[now]);
            now = fa[now];
        } else {
            int tar = size[ch[now][0]] > size[ch[now][1]] ? 0 : 1;
            if (cur[now] == -1)
                cur[now] = tar;
            else if (cur[now] == tar)
                cur[now] = cur[now] ^ 1;
            else {
                ll ans0 = 0, ans1 = 0;
                root[now] = mergeSeg(root[ch[now][0]], root[ch[now][1]], ans0, ans1);
                ans += min(ans0, ans1);
                now = fa[now];
                continue;
            }
            now = ch[now][cur[now]];
        }
    }
    return ans;
}
int main() {
    init();
    n = getnum();
    while (++tot) {
        int x = getnum();
        if (x == -1) break;
        int f = st[top];
        fa[tot] = f;
        if (f) {
            (ch[f][0] ? ch[f][1] : ch[f][0]) = tot;
            if (ch[f][1]) top--;
        }
        if (x) 
            value[tot] = x;
        else
            st[++top] = tot;
    } 
    tot--;
    dfs();
    printf("%lld\n", solve());
    return 0;
}
```