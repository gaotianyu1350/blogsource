title: "[Solution][BZOJ3545][BZOJ3551][ONTAK2010]Peaks"
date: 2015-01-11 21:31:01
tags: [BZOJ,splay,并查集,主席树,DFS序,倍增]
categories: 题解
---
话说这条好早以前就看了，现在才做
<!--more-->
## 题目
[传送门3545](http://www.lydsy.com/JudgeOnline/problem.php?id=3545)
[传送门3551](http://www.lydsy.com/JudgeOnline/problem.php?id=3551)

## 分析
对于`BZOJ3545`，因为不强制在线，我们可以将边和询问都按照权值排序。然后权值由小到打加边，维护并查集。对于每个联通块都维护一棵splay，查询穿插进行。不过用这种方法我T了一个点，不知道是splay挂了还是效率就是渣……最后用在线的方法水过去了。

强制在线的方法更为巧妙。仍然按照边的权值排序。初始时每个节点自己为一棵树，由小到大加边，每连接两棵不同的树，就将两棵树接在一个权值为边的权值的新的节点上，这个新的节点是新树的根。这样得到的一定是一棵二叉树，所有原图中的点在树中一定都是叶子节点，而且边的权值越往根上走越大。这样我们查询的时候从节点$v$倍增到权值小于等于$x$的最靠上的节点，则以这个节点为根的子树中包含的点就是当前能够到达的所有点，然后再用`DFS序`+`主席树`就可以查询`k`大了。

刚开始的时候没有考虑到整个图并不联通的情况，不过竟然还是`A`了。后来是在对拍的时候才发现了这个漏洞。

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <cmath>
#include <iostream>
#include <algorithm>
#include <queue>
using namespace std;
const int MAXN = 1e5 + 10;
const int MAXL = 2e5 + 10;
const int MAXM = 5e5 + 10;
const int MAXNODE = 23 * 1e5 + 10;
const int MAXB = 23;
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
struct edge { int x, y, w;
    void init() { x = getnum(); y = getnum(); w = getnum(); }
    bool operator < (const edge other) const { return w < other.w; }
}e[MAXM]; int weight[MAXN], duiying[MAXN][2], ch[MAXL][2];
int root[MAXN], size[MAXNODE], ll[MAXNODE], rr[MAXNODE], tot = 0;
int father[MAXL], f[MAXL][MAXB], sh[MAXN], height[MAXN], dfstot = 0, etot = 0;
int n, m, q; bool visit[MAXL] = {0};

inline void memcheck() {
    int tar = sizeof(e) + sizeof(weight) + sizeof(duiying) + sizeof(ch) + sizeof(ch) 
        + sizeof(root) + sizeof(size) + sizeof(ll) + sizeof(rr) + sizeof(father) 
        + sizeof(f) + sizeof(height);
    printf("%d\n", tar / 1024 / 1024);        
}
int getfather(int x) {
    if (x == father[x]) return x; return father[x] = getfather(father[x]);
}
void add(int &now, int pre, int l, int r, int value) {
    size[now = ++tot] = size[pre] + 1;
    if (l == r) return;
    int mid = (l + r) >> 1;
    if (value <= mid) add(ll[now], ll[pre], l, mid, value), rr[now] = rr[pre];
    else add(rr[now], rr[pre], mid + 1, r, value), ll[now] = ll[pre];
}
int find(int now, int pre, int l, int r, int k) {
    if (k > size[now] - size[pre]) return -1;
    if (l == r) return l;
    int mid = (l + r) >> 1;
    if (k <= size[ll[now]] - size[ll[pre]]) return find(ll[now], ll[pre], l, mid, k);
    else return find(rr[now], rr[pre], mid + 1, r, k - (size[ll[now]] - size[ll[pre]]));
}
inline void dfs(int now) {
    while (now) { visit[now] = true;
        if (ch[now][0]) {
            if (now > n) duiying[now - n][0] = dfstot + 1;
            f[ch[now][0]][0] = now; int old = now; now = ch[now][0];
            ch[old][0] = 0;
        } else if (ch[now][1]) {
            f[ch[now][1]][0] = now; int old = now; now = ch[now][1];
            ch[old][1] = 0;
        } else {
            if (now > n) duiying[now - n][1] = dfstot;
            else dfstot++, add(root[dfstot], root[dfstot - 1], 1, n, height[now]);
            now = f[now][0];
        }
    }
}
inline void initf() {
    for (int i = 1; i < MAXB; i++)
        for (int j = 1; j <= n + etot; j++)
            f[j][i] = f[f[j][i - 1]][i - 1];
}
inline int go(int now, int limit) {
    for (int i = MAXB - 1; i >= 0; i--)
        if (f[now][i] && weight[f[now][i] - n] <= limit) now = f[now][i];
    return now;
}
inline int solve(int v, int limit, int k) {
    int nowroot = go(v, limit);
    if (nowroot == v) {
        if (k == 1) return sh[height[v]]; else return -1;
    }
    int l = duiying[nowroot - n][0], r = duiying[nowroot - n][1];
    int ans = find(root[r], root[l - 1], 1, n, k);
    if (ans != -1) ans = sh[ans]; return ans;
}
bool cmp_big(const int &a, const int &b) { return a > b; }
int main() {
    n = getnum(); m = getnum(); q = getnum();
    for (int i = 1; i <= n; i++) height[i] = getnum(), sh[i] = height[i];
    sort(sh + 1, sh + 1 + n, cmp_big);
    for (int i = 1; i <= n; i++) height[i] = (int)(lower_bound(sh + 1, sh + 1 + n, height[i], cmp_big) - sh);
    for (int i = 1; i <= m; i++) e[i].init();
    sort(e + 1, e + 1 + m);
    for (int i = 1; i <= n * 2 - 1; i++) father[i] = i;
    for (int i = 1; i <= m && etot < n - 1; i++) {
        if (getfather(e[i].x) == getfather(e[i].y)) continue;
        weight[++etot] = e[i].w;
        int fx = father[e[i].x], fy = father[e[i].y];
        father[fx] = father[fy] = n + etot;
        ch[n + etot][0] = fx, ch[n + etot][1] = fy;
    } 
    for (int i = 1; i <= n; i++)
        if (!visit[getfather(i)]) dfs(getfather(i));
    int lastans = 0; initf();
    for (int i = 1; i <= q; i++) {
        int v, x, k; v = getnum(); x = getnum(); k = getnum();
        if (lastans != -1) v ^= lastans, x ^= lastans, k ^= lastans;
        printf("%d\n", lastans = solve(v, x, k));
    }
}
```