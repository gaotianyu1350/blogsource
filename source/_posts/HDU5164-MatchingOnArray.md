title: "[Solution][HDU5164][BestCoder#27 C]Matching on Array"
date: 2015-01-25 10:10:45
tags: [HDU,BestCoder,后缀数组]
categories: 题解
---
生疏了
<!--more-->
## 题目描述
[传送门](http://acm.hdu.edu.cn/showproblem.php?pid=5164)

## 分析
经典的一匹配多。但是题目中有个蛋疼的比例缩放的要求。这怎么搞？

刚开始以为这是一道和`BZOJ1461`一样的题目。结果发现没法用`KMP`。然后……然后考场上我就陷入了无助（果真还是太弱了）。

离比赛结束还有半个小时的时候才想到了靠谱的算法。既然原题中匹配的数列要求比例相同就好，那我们把数列转换成前一个数字与后一个数字的比值不就好了！这样就转换成了普通的一匹配多的问题。

这道题可以用`AC自动机`（需要用`MAP`来存储儿子）或者`后缀数组`。（看到还有神犇用`后缀树`和`分块+HASH`不能仰慕更多。）结果因为好久没写`后缀数组`了考场上也没有写完……看来要多复习基本算法了。

一些细节：存储比例可以用`double`也可以分数的最简形式。我用的是分数的最简形式。这样的话有一个问题需要处理，就是匹配的时候只能匹配$0,2,4...$等起始位置开头的后缀，因为不能将二元组拆开。这里需要特殊处理一下。

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
using namespace std;
typedef long long ll;
const int MAXN = 1e5 + 10;
const int MAXNODE = 2e5 + 10;
const int MAXK = 3e5 + 10;
int n, m, k;
int a[MAXN], b[MAXK], bs[MAXK * 2];
int s[MAXNODE], sa[MAXNODE], rank[MAXNODE], t1[MAXNODE], t2[MAXNODE], cnt[MAXNODE];
int anscnt[MAXNODE];
inline int gcd(int a, int b) {
    if (!b) return a;
    else return gcd(b, a % b);
}
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
inline void makeSA(int n) {
    int m = 10001; int *x = t1, *y = t2;
    memset(t1, 0, sizeof(t1));
    memset(t2, 0, sizeof(t2));
    memset(cnt, 0, sizeof(cnt));
    for (int i = 0; i < n; i++) x[i] = s[i];
    for (int i = 0; i < n; i++) cnt[x[i]]++;
    for (int i = 1; i < m; i++) cnt[i] += cnt[i - 1];
    for (int i = n - 1; i >= 0; i--) sa[--cnt[x[i]]] = i;
    for (int t = 1; t < n; t <<= 1) { int tot = 0;
    for (int i = n - t; i < n; i++) y[tot++] = i;
    for (int i = 0; i < n; i++) if (sa[i] >= t) y[tot++] = sa[i] - t;
    for (int i = 0; i < m; i++) cnt[i] = 0;
    for (int i = 0; i < n; i++) cnt[x[y[i]]]++;
    for (int i = 1; i < m; i++) cnt[i] += cnt[i - 1];
    for (int i = n - 1; i >= 0; i--) sa[--cnt[x[y[i]]]] = y[i];
    swap(x, y); m = 1; x[sa[0]] = 0;
    for (int i = 1; i < n; i++)
        x[sa[i]] = y[sa[i]] == y[sa[i - 1]] && y[sa[i] + t] == y[sa[i - 1] + t] ? m - 1 : m++; 
    }
}
inline int scmp(int *a, int *b, int len1, int len2) {
    int tar = min(len1, len2);
    for (int i = 0; i < tar; i++) {
        if (a[i] < b[i]) return -1;
        if (a[i] > b[i]) return 1;
    }
    if (len1 == len2) return 0;
    if (len1 < len2) return -1; else return 1;
}
inline int scmp2(int *a, int *b, int len1, int len2) {
    int tar = min(len1, len2);
    for (int i = 0; i < tar; i++) {
        if (a[i] < b[i]) return -1;
        if (a[i] > b[i]) return 1;
    } return 0;
}
int main() {
    freopen("input.txt", "r", stdin);
    int t = getnum();
    while (t--) {
        n = getnum(); m = getnum(); int tot = 0;
        for (int i = 1; i <= n; i++) {
            a[i] = getnum();
            if (i > 1) {
                int tmp = gcd(a[i], a[i - 1]);
                s[tot++] = a[i - 1] / tmp;
                s[tot++] = a[i] / tmp;
            }
        }
        makeSA(tot); ll ans = 0;
        memset(anscnt, 0, sizeof(anscnt));
        anscnt[0] = sa[0] % 2 == 0 ? 1 : 0;
        for (int i = 1; i < tot; i++)
            anscnt[i] = anscnt[i - 1] + (sa[i] % 2 == 0 ? 1 : 0);
        for (int i = 1; i <= m; i++) {
            k = getnum(); int tmptot = 0;
            for (int j = 1; j <= k; j++) {
                b[j] = getnum();
                if (j > 1) {
                    int tmp = gcd(b[j], b[j - 1]);
                    bs[tmptot++] = b[j - 1] / tmp;
                    bs[tmptot++] = b[j] / tmp;
                }
            }
            if (k == 1) { ans += n; continue; }
            int l, r, ans1, ans2;
            l = 0, r = tot - 1;
            while (l < r) {
                int mid = (l + r) >> 1;
                if (scmp(s + sa[mid], bs, tot - sa[mid], tmptot) >= 0)
                    r = mid;
                else l = mid + 1;
            } ans1 = l;
            l = 0, r = tot - 1;
            while (l < r) {
                int mid = (l + r + 1) >> 1;
                if (scmp2(s + sa[mid], bs, tot - sa[mid], tmptot) <= 0)
                    l = mid;
                else r = mid - 1;
            } ans2 = l;
            if (!scmp2(s + sa[ans1], bs, tot - sa[ans1], tmptot)
             && !scmp2(s + sa[ans2], bs, tot - sa[ans2], tmptot))
                ans += anscnt[ans2] - (!ans1 ? 0 : anscnt[ans1 - 1]);
        } printf("%I64d\n", ans);
    }
}
```