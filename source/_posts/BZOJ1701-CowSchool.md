title: "[Solution][BZOJ1701][USACO2007 Jan]Cow School"
date: 2015-01-28 14:43:43
tags: [BZOJ,USACO,鸟哥神题系列,CDQ分治]
categories: 题解
---
原来是分治
<!--more-->
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1701)

## 分析
既然老师要删除前$d$小个，那我们先按照得分率$F_i$排序。

老师计算出来的
$$G=\frac{\sum\_{i=d+1}^{i\leq n}T\_i}{\sum\_{i=d+1}^{i\leq n}P\_i}$$
变形一下就可以得到
$$ \sum\_{i=d+1}^{i\leq n}T\_i-GP\_i=0 $$
只要我们能找到一个方法，替换几次考试，使得上面这一坨大于$0$，贝茜的目的就达到了。显然只有当$Max\{T\_i-GP\_i\},1\leq i\leq d$大于$Min\{T\_i-GP\_i\},d+1\leq i\leq n$时，这个$d$才能够被计入答案。

这样问题就转换为了求对于区间$1..j$，求$Max\{T\_i-G\_{j+1}P\_i\},1\leq i\leq j$和对于区间$j..n$，求$Min\{T\_i-G\_jP\_i\},j+1\leq i\leq n$。可以看到目标函数是截距的形式，我们可以维护凸壳来求。又因为斜率和坐标都不单调，所以可以用`CDQ分治`。

再来复习一下`CDQ分治`的步骤

定义`Solve(l,r)`解决$\[l,r\]$之间的点对$\[l,r\]$之间询问的贡献。
①、`Solve(l,mid)`
②、计算$\[l,mid\]$对$\[mid+1,r\]$的贡献
③、`Solve(mid+1,r)`

在这里首先按照$G\_i$排序（因为查询的时候斜率单调才能够做到$O(n)$）。在`Solve(l,r)`的开始，把所有分数率小的元素放到左边，分数率大的元素放到右边（因为只有分数率小的元素可以更新分数率大的元素的答案）。在①之后②之前，计算$\[l,mid\]$之间的凸包。在③之后，归并排序为按照$P\_i$升序。

好吧这里讲的比较混乱详见代码。分治的方法和`Cash`一题很类似。

分治部分还是比较好写的。但是我犯了几个逗比错误结果调试了一上午。一个是用栈的时候没有分清楚自己的$top$指针是指的最后一个元素还是最后一个元素的后一个元素。再一个是排序的时候关键字搞错了。所以以后写完这种比较长的程序应该通读一下自己的代码。

## 代码
``` c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;
typedef long long ll;
const double eps = 1e-10;
const double inf = 1e10;
const int MaxN = 5e4 + 10;
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
struct tdata {
    int x, y, pos, loc;
    double rate, k, kk;
    void get() {
        y = getnum(); x = getnum();
        rate = (double)y / x;
    }
    bool operator < (const tdata &o) const {
        return x < o.x || (x == o.x && y < o.y);
    }
    double calc(double kk) {
        return y - kk * x; 
    }
} t[MaxN], tmp[MaxN];
int n, s[MaxN];
double fMin[MaxN], fMax[MaxN];
bool cmp_rate_max(const tdata &a, const tdata &b) {
    return a.rate < b.rate;
}
bool cmp_rate_min(const tdata &a, const tdata &b) {
    return a.rate > b.rate;
}
bool cmp_min(const tdata &a, const tdata &b) {
    return a.k < b.k;
}
bool cmp_max(const tdata &a, const tdata &b) {
    return a.kk > b.kk;
}
inline ll cross(const tdata &a, const tdata &b, const tdata &c) {
    return (ll)(b.x - a.x) * (c.y - b.y) - (ll)(c.x - b.x) * (b.y - a.y);
}
void Solve_min(int l, int r) {
    if (l == r) {
        fMin[t[l].loc] = min(fMin[t[l].loc], t[l].calc(t[l].k));
        return;
    }
    int mid = (l + r) >> 1, pl = l, pr = mid + 1;
    for (int i = l; i <= r; i++)
        (t[i].pos <= mid ? tmp[pl++] : tmp[pr++]) = t[i];
    memcpy(t + l, tmp + l, sizeof(tdata) * (r - l + 1));
    Solve_min(l, mid);
    int tot = 0;
    for (int i = l; i <= mid; i++) {
        while (tot >= 2 && cross(t[s[tot - 1]], t[s[tot]], t[i]) <= 0)
            tot--;
        s[++tot] = i;
    } 
    int head = 1;
    for (int i = mid + 1; i <= r; i++) {
        while (head < tot && t[s[head + 1]].calc(t[i].k) < t[s[head]].calc(t[i].k))
            head++;
        fMin[t[i].loc] = min(fMin[t[i].loc], t[s[head]].calc(t[i].k));
    }
    Solve_min(mid + 1, r);
    merge(t + l, t + mid + 1, t + mid + 1, t + r + 1, tmp + l);
    memcpy(t + l, tmp + l, sizeof(tdata) * (r - l + 1));
}
void Solve_max(int l, int r) {
    if (l == r) {
        fMax[t[l].loc] = max(fMax[t[l].loc], t[l].calc(t[l].kk));
        return;
    }
    int mid = (l + r) >> 1, pl = l, pr = mid + 1;
    for (int i = l; i <= r; i++)
        (t[i].pos <= mid ? tmp[pl++] : tmp[pr++]) = t[i];
    memcpy(t + l, tmp + l, sizeof(tdata) * (r - l + 1));
    Solve_max(l, mid);
    int tot = 0;
    for (int i = l; i <= mid; i++) {
        while (tot >= 2 && cross(t[s[tot - 1]], t[s[tot]], t[i]) >= 0)
            tot--;
        s[++tot] = i;
    }
    int head = 1;
    for (int i = mid + 1; i <= r; i++) {
        while (head < tot && t[s[head + 1]].calc(t[i].kk) > t[s[head]].calc(t[i].kk))
            head++;
        fMax[t[i].loc] = max(fMax[t[i].loc], t[s[head]].calc(t[i].kk));
    }
    Solve_max(mid + 1, r);
    merge(t + l, t + mid + 1, t + mid + 1, t + r + 1, tmp + l);
    memcpy(t + l, tmp + l, sizeof(tdata) * (r - l + 1));
}
inline int dcmp(const double &a, const double &b) {
    if (fabs(a - b) < eps) return 0;
    return a > b ? 1 : -1;
}
int main() {
    freopen("input.txt", "r", stdin);
    n = getnum();
    for (int i = 1; i <= n; i++)
        t[i].get();
    sort(t + 1, t + 1 + n, cmp_rate_max);
    int sumX = 0, sumY = 0;
    for (int i = n; i >= 1; i--) {
        sumX += t[i].x;
        sumY += t[i].y;
        t[i].k = (double)sumY / sumX;
        t[i - 1].kk = t[i].k;
        t[i].loc = i;
        fMin[i] = inf; fMax[i] = -inf;
    }
    sort(t + 1, t + 1 + n, cmp_rate_max);
    for (int i = 1; i <= n; i++)
        t[i].pos = i;
    sort(t + 1, t + 1 + n, cmp_max);
    Solve_max(1, n);
    sort(t + 1, t + 1 + n, cmp_rate_min);
    for (int i = 1; i <= n; i++)
        t[i].pos = i;
    sort(t + 1, t + 1 + n, cmp_min);
    Solve_min(1, n);
    static vector<int> ans;
    for (int i = 1; i < n; i++) {
        if (dcmp(fMax[i], fMin[i + 1]) > 0)
            ans.push_back(i);
    }
    printf("%d\n", ans.size());
    for (int i = 0; i < (int)ans.size(); i++)
        printf("%d\n", ans[i]);
}
```