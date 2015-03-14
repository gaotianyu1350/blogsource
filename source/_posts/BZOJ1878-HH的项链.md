title: "[Solution][BZOJ1878][SDOI2009]HH的项链"
date: 2015-01-24 08:34:19
tags: [BZOJ,SDOI,树状数组,主席树,离线]
categorie: 题解
---
求区间内相同颜色的个数
<!--more-->
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1878)

## 分析
在线做法：主席树……

离线做法：好吧我的第一反应是莫队……

当然还有比莫队更快的做法。思考一下如果我们要用某数据结构维护区间查询的时候，每种颜色只能标记一个点。假设我们先都标记第一个点。这样如果只查询左端点为$1$的询问自然没有问题。但是随着左端点的增长，显然只标记每种颜色的第一个点是不可以的。

于是离线的做法来了。对所有询问按照左端点排序。对序列上的每个点，记录和它同种颜色的下一个点的位置$nxt$。随着左端点位置的右移，左边的点已经不可能再出现在询问里面了。所以将这些点的$nxt$标记。树状数组查询即可。

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
using namespace std;
const int MAXN = 5e4 + 10;
const int MAXM = 2e5 + 10;
const int MAXC = 1e6 + 10;
int nxt[MAXN], first[MAXC], tree[MAXN], n, m;
int a[MAXN];
struct querydata {
    int l, r, loc, ans;
    bool operator < (const querydata other) const {
        return l < other.l; 
    }
} q[MAXM];
inline void add(int x, int v) {
    for (int i = x; i <= n; i += i & -i)
        tree[i] += v;
}
inline int get(int x) { int ans = 0;
    if (!x) return 0;
    for (int i = x; i > 0; i -= i & -i)
        ans += tree[i];
    return ans;
}
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
bool cmp(const querydata &a, const querydata &b) {
    return a.loc < b.loc;
}
int main() {
    n = getnum(); int maxcolor = 0;
    for (int i = 1; i <= n; i++) 
        a[i] = getnum(), maxcolor = max(maxcolor, a[i]);
    for (int i = n; i >= 1; i--)
        nxt[i] = first[a[i]], first[a[i]] = i;
    for (int i = 1; i <= maxcolor; i++)
        if (first[i]) add(first[i], 1);
    m = getnum();
    for (int i = 1; i <= m; i++)
        q[i].l = getnum(), q[i].r = getnum(), q[i].loc = i;
    sort(q + 1, q + 1 + m);
    int last = 1;
    for (int i = 1; i <= m; i++) {
        for (int j = last; j < q[i].l; j++) {
            first[a[j]] = nxt[j];
            if (first[a[j]])
                add(first[a[j]], 1);
        } last = q[i].l;
        q[i].ans = get(q[i].r) - get(q[i].l - 1);
    }
    sort(q + 1, q + 1 + m, cmp);
    for (int i = 1; i <= m; i++)
        printf("%d\n", q[i].ans);
}
```
