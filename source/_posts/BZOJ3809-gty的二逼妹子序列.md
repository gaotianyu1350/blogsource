title: "[Solution][BZOJ3809]gty的二逼妹子序列"
date: 2015-03-27 15:54:13
tags: [BZOJ,莫队,权值分块]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3809)

## 分析
题目要求复杂度在$O(nsqrt n)$左右。如果直接在线貌似没法搞……所以先上莫队。

莫队的修改次数是$O(nsqrt n)$，查询次数$O(n)$。也就是说我们需要一个修改$O(1)$，查询$O(sqrt n)$的数据结构。线段树pass掉。于是就get到新技能——`权值分块`。

将权值分块后，对每个值维护计数器，每个块维护种类数。这样就满足复杂度的要求了。

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

const int MaxN = 1e5 + 10;
const int MaxM = 1e6 + 10;
const int MaxSize = 400;
const int MaxBlock = MaxN / MaxSize + 10;

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

int bSt[MaxBlock], bEnd[MaxBlock], idx[MaxN], totBlock;
int value[MaxN];
int n, m;

struct query {
    int L, R, A, B, loc, ans;
    void get() {
        L = getnum(); R = getnum();
        A = getnum(); B = getnum();
    }

    bool operator < (const query &o) const {
        return idx[L] < idx[o.L] || (idx[L] == idx[o.L] && R < o.R);
    }
} q[MaxM];

namespace bvalue {
    int cnt[MaxN], kind[MaxBlock];

    inline void add(int x, int v) {
        cnt[x] += v;
        if (v == 1 && cnt[x] == 1)
            kind[idx[x]]++;
        else if (v == -1 && !cnt[x])
            kind[idx[x]]--;
    }

    inline int query(int l, int r) {
        int lb = idx[l];
        int rb = idx[r];
        if (l != bSt[lb]) lb++;
        if (r != bEnd[rb]) rb--;
        int ans = 0;

        if (lb <= rb) {
            for (int i = lb; i <= rb; i++)
                ans += kind[i];
            for (int i = l; i < bSt[lb]; i++)
                if (cnt[i]) ans++;
            for (int i = bEnd[rb] + 1; i <= r; i++)
                if (cnt[i]) ans++;
        } else {
            for (int i = l; i <= r; i++)
                if (cnt[i]) ans++;
        }

        return ans;
    }

    inline void clear() {
        memset(cnt, 0, sizeof(cnt));
        memset(kind, 0, sizeof(kind));
    }
}

inline void init() {
    totBlock = 0;
    for (int i = 1; i <= n; i += MaxSize) {
        ++totBlock;
        bSt[totBlock] = i;
        bEnd[totBlock] = min(n, i + MaxSize - 1);
        for (int j = bSt[totBlock]; j <= bEnd[totBlock]; j++)
            idx[j] = totBlock;
    }
}

bool cmp(const query &a, const query &b) {
    return a.loc < b.loc;
}

int main() {
    n = getnum(); m = getnum();
    for (int i = 1; i <= n; i++) value[i] = getnum();
    for (int i = 1; i <= m; i++) {
        q[i].get();
        q[i].loc = i;
    }
    init();
    sort(q + 1, q + 1 + m);
    
    for (int i = 1; i <= m; i++) {
        if (i == 1 || idx[q[i - 1].L] != idx[q[i].L]) {
            bvalue::clear();
            for (int j = q[i].L; j <= q[i].R; j++)
                bvalue::add(value[j], 1);
            q[i].ans = bvalue::query(q[i].A, q[i].B);
        } else {
            if (q[i - 1].L < q[i].L) {
                for (int j = q[i - 1].L; j < q[i].L; j++)
                    bvalue::add(value[j], -1);
            } else {
                for (int j = q[i].L; j < q[i - 1].L; j++)
                    bvalue::add(value[j], 1);
            }

            for (int j = q[i - 1].R + 1; j <= q[i].R; j++)
                bvalue::add(value[j], 1);
            q[i].ans = bvalue::query(q[i].A, q[i].B);
        }
    }

    sort(q + 1, q + 1 + m, cmp);
    for (int i = 1; i <= m; i++)
        printf("%d\n", q[i].ans);
}
```