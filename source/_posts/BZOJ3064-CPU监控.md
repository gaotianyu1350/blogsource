title: "[Solution][BZOJ3064][TYVJ1518]CPU监控"
date: 2015-03-05 17:39:49
tags: [BZOJ,线段树,TYVJ]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3064)

## 分析
一道巨恶心的线段树标记题，写完这道题才发现自己根本不会线段树……

看似只是普通的区间修改、区间加、区间查询最大，但是加了一个操作：区间查询历史最大。本来以为只要在原有的修改标记、加标记和最大标记之外加一个历史最大标记即可。但是有一种特殊情况不能处理：如果有一个标记还没有下传，就被另一个标记覆盖了，而询问的时候要深入到最下面询问，那么就无法获得最大值了。

显然我们需要再加三个标记：历史最大标记$hisMax$，历史最大修改标记$hisEqu$和历史最大加标记$hisAdd$。写线段树标记最重要的是搞清楚标记的含义和加入的时间点。

正常的线段树标记都是表示对子树的操作。如果一个标记存在，说明对自己的修改已经进行完了（也就是说自己的状态一定是最新的），但是子树一定还没有更新，而且标记下传后都要清空。那我们的$hisEqu$和$hisAdd$也是这样，并且为了不和原来的标记搞混，我们要求$hisEqu$和$hisAdd$不能包括最后的操作。只有当原来的修改和加标记变化的时候，才用它们去更新$hisEqu$和$hisAdd$。

需要特别注意的是，$hisEqu$和$hisAdd$的时间在子树里面的标记和当前节点的标记之间。也就是说，它建立在子树原有的值和子树的标记的基础上，但是不受自身节点标记的影响。所以在`pushdown()`的时候，先`pushdown()`$hisEqu$和$hisAdd$，而且同时要把子树中的标记与$hisEqu$与$hisAdd$结合（强制，因为它们建立在子树标记的基础上），然后再下传一般标记。

还有一些细节：$hisEqu$和$hisAdd$是两种不同的情况。$hisEqu$是赋值，之后可能有加，而$hisAdd$是只有加，没有赋值。这两种在不同情况下孰优孰劣不一样，所以要分开讨论。还有就是一些重复的过程尽量写成函数，保证思维的清晰。

更多细节看代码吧……

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
 
const int MaxN = 1e5 + 100;
const int MaxNode = 4e5 + 100;
const int inf = ((unsigned int)1 << 31) - 1;
 
bool hasch[MaxNode];
int maxx[MaxNode], hmax[MaxNode];
int delta_ch[MaxNode], delta_add[MaxNode];
int his_ch[MaxNode], his_add[MaxNode];
int value[MaxN], n, m;
 
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
 
inline char getmychar() {
    char c;
    while (!((c = getchar()) >= 'A' && c <= 'Z'));
    return c;
}
 
inline void update(int now) {
    maxx[now] = max(maxx[now << 1], maxx[(now << 1) + 1]);
    hmax[now] = max(hmax[now], max(hmax[now << 1], hmax[(now << 1) + 1]));
    hmax[now] = max(hmax[now], maxx[now]);
}
 
inline void update(int &a, int b) {
    a = max(a, b);
}
 
inline void changeHis(int now) {
    if (hasch[now])
        update(his_ch[now], delta_ch[now]);
    else
        update(his_add[now], delta_add[now]);
}
 
inline void pushCH(int now, int v) {    
    maxx[now] = v;
    hmax[now] = max(hmax[now], maxx[now]);
     
    changeHis(now);
    delta_ch[now] = v; hasch[now] = true;
    delta_add[now] = 0;
}
 
inline void pushADD(int now, int v) {
    maxx[now] += v;
    hmax[now] = max(hmax[now], maxx[now]);
     
    changeHis(now);
    delta_add[now] += v;    
    if (hasch[now]) {
        delta_ch[now] += delta_add[now];
        delta_add[now] = 0;
    }   
}
 
inline void pushdown(int now) {
    int l = now << 1, r = (now << 1) + 1;
     
    int l_ch = his_ch[now];
    int r_ch = his_ch[now];
    int l_add = maxx[l] + his_add[now]; 
    int r_add = maxx[r] + his_add[now];
     
    update(hmax[l], max(l_ch, l_add));
    update(hmax[r], max(r_ch, r_add));
     
    update(his_ch[l], his_ch[now]);
    update(his_ch[r], his_ch[now]);
    if (hasch[l])
        update(his_ch[l], delta_ch[l] + his_add[now]);
    else
        update(his_add[l], delta_add[l] + his_add[now]);
    if (hasch[r])
        update(his_ch[r], delta_ch[r] + his_add[now]);
    else
        update(his_add[r], delta_add[r] + his_add[now]);
     
    his_ch[now] = -inf;
    his_add[now] = 0;
     
    if (hasch[now]) {
        pushCH(l, delta_ch[now]);
        pushCH(r, delta_ch[now]);
    }
     
    if (delta_add[now] != 0) {
        pushADD(l, delta_add[now]);
        pushADD(r, delta_add[now]);
    }
     
    delta_ch[now] = 0; hasch[now] = false;
    delta_add[now] = 0;
}

void build(int now, int l, int r) {
    hmax[now] = -inf;
    his_ch[now] = -inf;
    if (l == r) {
        hmax[now] = maxx[now] = value[l];
        return;
    }
    int mid = (l + r) >> 1;
    build(now << 1, l, mid);
    build((now << 1) + 1, mid + 1, r);
    update(now);
}
 
void add(int now, int l, int r, int x, int y, int v) {
    if (x <= l && r <= y) {
        pushADD(now, v);
        return;
    }
    pushdown(now);
    int mid = (l + r) >> 1;
    if (x <= mid) add(now << 1, l, mid, x, y, v);
    if (y > mid) add((now << 1) + 1, mid + 1, r, x, y, v);
    update(now);
}
 
void change(int now, int l, int r, int x, int y, int v) {
    if (x <= l && r <= y) {
        pushCH(now, v);
        return;
    }
    pushdown(now);
    int mid = (l + r) >> 1;
    if (x <= mid) change(now << 1, l, mid, x, y, v);
    if (y > mid) change((now << 1) + 1, mid + 1, r, x, y, v);
    update(now);
}
 
int query(int *themax, int now, int l, int r, int x, int y) {
    if (x <= l && r <= y) return themax[now];
    pushdown(now);
    int mid = (l + r) >> 1, ans = -inf;
    if (x <= mid) 
        ans = max(ans, query(themax, now << 1, l, mid, x, y));
    if (y > mid)
        ans = max(ans, query(themax, (now << 1) + 1, mid + 1, r, x, y));
    return ans;
}
 
int main() {
    n = getnum();
    for (int i = 1; i <= n; i++) value[i] = getnum();
    build(1, 1, n);
     
    m = getnum();
    while (m--) {
        char intype = getmychar();
        int x, y, z;
        switch (intype) {
            case 'Q':
                x = getnum(); y = getnum();
                if (x > y) swap(x, y);
                printf("%d\n", query(maxx, 1, 1, n, x, y));
                break;
            case 'A':
                x = getnum(); y = getnum();
                if (x > y) swap(x, y);
                printf("%d\n", query(hmax, 1, 1, n, x, y));
                break;
            case 'P':
                x = getnum(); y = getnum(); z = getnum();
                if (x > y) swap(x, y);
                add(1, 1, n, x, y, z);
                break;
            case 'C':
                x = getnum(); y = getnum(); z = getnum();
                if (x > y) swap(x, y);
                change(1, 1, n, x, y, z);
                break;
        }
    }
}
```