title: "[Solution][BZOJ2741]L"
date: 2015-03-05 17:47:07
tags: [BZOJ,分块,可持久化Trie,xor]
categories: 题解
---

## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=2741)

## 分析
首先区间`xor`和可以转为两个前缀和的`xor`值。

如果是区间查询对于一个数的最大`xor`值可以直接用`可持久化Trie`，但数对就比较难办了。还好有分块大法。

令$Best_{i,j}$表示从第$i$块到第$j$块的区间内，最大的`xor`和。可以在$O(n\sqrt n~logn)$的时间内解决（枚举第一个数，枚举右端点的块，用`可持久化Trie`区间查询）。对于询问，中间的整块部分直接用$Best$回答，边上零碎的用`可持久化Trie`暴力搞一搞就可以了。

<!--more-->
## 代码
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
using namespace std;
 
typedef long long ll;
typedef unsigned int ui;
 
const int MaxBit = 31;
const int MaxN = 1.2e4 + 100;
const int MaxNode = 34 * 1.2e4 + 100;
const int MaxBlock = 130;
const int BLOCK = 120;
 
ll value[MaxN], sum[MaxN], lastAns = 0;
ll best[MaxBlock][MaxBlock];
int start[MaxBlock], end[MaxBlock];
int inb[MaxN];
bool isstart[MaxN], isend[MaxN];
 
int size[MaxNode], ch[MaxNode][2], root[MaxN], tot = 0;
int n, m, cntb;
 
inline void mck() {
    int tar = sizeof(value) * 2 + sizeof(best) + sizeof(start) * 2 + sizeof(inb) + sizeof(isstart) * 2;
    tar += sizeof(size) + sizeof(ch) + sizeof(root);
    printf("%d\n", tar / 1000000);
}
 
inline int add(int pre, ll v) {
    int now = ++tot;
    int res = now;
    for (int i = MaxBit; i >= 0; i--) {
        size[now] = size[pre] + 1;
        if (v >> i & 1) {
            ch[now][1] = ++tot;
            ch[now][0] = ch[pre][0];
            now = ch[now][1]; pre = ch[pre][1];
        } else {
            ch[now][0] = ++tot;
            ch[now][1] = ch[pre][1];
            now = ch[now][0]; pre = ch[pre][0];
        }
    }
    size[now] = size[pre] + 1;
    return res;
}
 
inline ll query(int pre, int now, ll v) {
    ll ans = 0;
    for (int i = MaxBit; i >= 0; i--) {
        int dir = (v >> i & 1) ? 0 : 1;
        int tmp = size[ch[now][dir]] - size[ch[pre][dir]];
        if (tmp) {
            ans += 1LL << i;
            now = ch[now][dir];
            pre = ch[pre][dir];
        } else {
            now = ch[now][dir ^ 1];
            pre = ch[pre][dir ^ 1];
        }
    }
    return ans;
}
 
inline ll solve(int x, int y, ll v) {
    if (!x)
        return max(v, query(root[0], root[y], v));
    else
        return query(root[x - 1], root[y], v);
}
 
inline ll getnum() {
    ll ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
 
int main() {
 
    n = getnum(); m = getnum();
    for (int i = 1; i <= n; i++) {
        value[i] = getnum();
        sum[i] = sum[i - 1] ^ value[i];
        root[i] = add(root[i - 1], sum[i]);
    }
     
    cntb = n / BLOCK + (n % BLOCK ? 1 : 0);
    for (int i = 1; i <= cntb; i++) {
        start[i] = (i - 1) * BLOCK + 1;
        end[i] = min(n, i * BLOCK);
        isstart[start[i]] = true;
        isend[end[i]] = true;
    }
    for (int i = 1; i <= n; i++)
        inb[i] = i / BLOCK + (i % BLOCK ? 1 : 0);
 
    for (int i = 1; i <= n; i++) {
        int curb = inb[i];
        for (int j = curb; j <= cntb; j++) {
            if (j > curb) 
                best[curb][j] = max(best[curb][j], best[curb][j - 1]);
            best[curb][j] = max(best[curb][j], solve(start[j], end[j], sum[i]));
        }
    }
     
    while (m--) {
        ll x = getnum(); ll y = getnum();
        x = (x + lastAns) % n + 1;
        y = (y + lastAns) % n + 1;
        if (x > y) swap(x, y);
        int beginX = x - 1, beginY = y - 1;
         
        lastAns = 0;
        int laststart = x;
        int firstend = beginY;
         
        if (x <= beginY) {
            int startB = inb[x];
            int endB = inb[beginY];
            if (!isstart[x]) startB++;
            if (!isend[beginY]) endB--;
            if (startB <= endB) {
                laststart = max(laststart, end[endB] + 1);
                firstend = min(firstend, start[startB] - 1);
                 
                for (int i = startB; i <= endB; i++)
                    lastAns = max(lastAns, best[i][endB]);
                for (int i = laststart; i <= y; i++)
                    lastAns = max(lastAns, solve(beginX, beginY, sum[i]));
            }
        }
         
        for (int i = beginX; i <= firstend; i++) 
            lastAns = max(lastAns, solve(x, y, sum[i]));
             
        printf("%lld\n", lastAns);
    }
}
```