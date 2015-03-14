title: "[Solution][BZOJ3879]SvT"
date: 2015-02-05 11:26:31
tags: [BZOJ,后缀数组,RMQ]
categories: 题解
---
<!--more-->
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3879)

## 分析
和`BZOJ3238`类似，只不过那个题目里面要求的是所有后缀两两之间的`LCP`之和，而这里求的是给定的后缀，所以还要用`RMQ`。用`RMQ`的时候要小心数组越接界的情况。

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
using namespace std;
typedef long long ll;
const ll mod = 23333333333333333;
const int MaxN = 5e5 + 10;
const int MaxT = 3e6 + 10;
const int MaxB = 21;
int sa[MaxN], rank[MaxN], height[MaxN], n, m, maxbit;
int t1[MaxN], t2[MaxN], cnt[MaxN];
int rmq[MaxN][MaxB];
char s[MaxN];
struct qdata {
    int val, cnt;
    qdata(int v = 0, int c = 0)
        : val(v), cnt(c) {}
} st[MaxN];
int top, tmp[MaxT];
inline bool sacmp(int *y, int a, int b, int t) {
    int ranka1 = y[a];
    int rankb1 = y[b];
    int ranka2 = a + t < n ? y[a + t] : -1;
    int rankb2 = b + t < n ? y[b + t] : -1;
    return ranka1 == rankb1 && ranka2 == rankb2;
}
inline void makeSA() {
    int m = 26;
    int *x = t1, *y = t2;
    for (int i = 0; i < m; i++) cnt[i] = 0;
    for (int i = 0; i < n; i++) cnt[x[i] = s[i] - 'a']++;
    for (int i = 1; i < m; i++) cnt[i] += cnt[i - 1];
    for (int i = n - 1; i >= 0; i--)
        sa[--cnt[x[i]]] = i;
    for (int t = 1; t < n; t <<= 1) {
        int p = 0;
        for (int i = n - t; i < n; i++) y[p++] = i;
        for (int i = 0; i < n; i++)
            if (sa[i] >= t) y[p++] = sa[i] - t;
        for (int i = 0; i < m; i++) cnt[i] = 0;
        for (int i = 0; i < n; i++) cnt[x[y[i]]]++;
        for (int i = 1; i < m; i++) cnt[i] += cnt[i - 1];
        for (int i = n - 1; i >= 0; i--)
            sa[--cnt[x[y[i]]]] = y[i];
        swap(x, y);
        x[sa[0]] = 0; m = 1;
        for (int i = 1; i < n; i++)
            x[sa[i]] = sacmp(y, sa[i], sa[i - 1], t) ? m - 1 : m++;
    }
}
inline void makeHeight() {
    for (int i = 0; i < n; i++)
        rank[sa[i]] = i;
    height[n - 1] = 0; int k = 0;
    for (int i = 0; i < n; i++) {
        if (rank[i] == n - 1) continue;
        if (k) k--;
        int j = sa[rank[i] + 1];
        while (s[i + k] == s[j + k]) k++;
        height[rank[i]] = k;
    }
}
inline void makeRMQ() {
    maxbit = (int)(log(n) / log(2) + 1);
    for (int j = 0; j <= maxbit; j++) {
        int len = (1 << (j - 1));
        for (int i = 0; i < n; i++) {
            if (!j)
                rmq[i][j] = height[i];
            else
                if (i + len < n)
                    rmq[i][j] = min(rmq[i][j - 1], rmq[i + len][j - 1]);
        }
    }
}
inline int query(int x, int y) {
    int len = (int)(log(y - x) / log(2));
    return min(rmq[x][len], rmq[y - (1 << len)][len]);
}
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
bool cmp(const int &a, const int &b) {
    return rank[a] > rank[b];
}
int main() {
    n = getnum(); m = getnum();
    scanf("%s", s);
    makeSA();
    makeHeight();
    makeRMQ();
    while (m--) {
        int t = getnum();
        for (int i = 1; i <= t; i++)
            tmp[i] = getnum() - 1;
        sort(tmp + 1, tmp + 1 + t, cmp);
        t = unique(tmp + 1, tmp + 1 + t) - (tmp + 1);
        ll ans = 0, sum = 0; top = 0;
        for (int i = 2; i <= t; i++) {
            int cur = rank[tmp[i]];
            int pre = rank[tmp[i - 1]];
            qdata now(query(cur, pre), 1);
            while (top && st[top].val > now.val) {
                now.cnt += st[top].cnt;
                sum -= (ll)st[top].val * st[top].cnt;
                top--;
            }
            st[++top] = now;
            sum += (ll)now.val * now.cnt;
            ans += sum;
            if (ans >= mod) ans -= mod;
        }
        printf("%lld\n", ans);
    }
    return 0;
}
```