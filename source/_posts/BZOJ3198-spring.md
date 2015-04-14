title: "[Solution][BZOJ3198][SDOI2013]spring"
date: 2015-03-26 21:51:48
tags: [BZOJ,SDOI,Hash,容斥原理]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3198)

## 分析
$k$非常小，所以我们可以枚举有哪些位上是相同的。如果知道了是哪些位，我们就可以通过`Hash`计算个数。还有一个问题，就是以上计算出的是至少这些位上相同的个数，恰好要再通过容斥原理计算。

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int MaxN = 1e5 + 10;
const int MaxS = (1 << 6);

namespace hash {
    const ll MOD1 = 243403;
    const ll MOD2 = 1055897;
    const ll MOD3 = 9023431;
    const ll P1 = (1 << 30);
    const ll P2 = (1 << 30) + 233;
    const ll P3 = (1 << 30) + 6311;

    int point[MOD1], nxt[MaxN], v1[MaxN], v2[MaxN], cnt[MaxN];
    int tot;

    inline void clear() {
        memset(point, 0, sizeof(point));
        tot = 0;
    }

    inline int calcHash1(int *a, int s) {
        ll ans = 0;
        for (int i = 0; i < 6; i++, s >>= 1)
            if (s & 1)
                ans = (ans * P1 % MOD1 + a[i] % MOD1) % MOD1;
        return ans;
    }

    inline int calcHash2(int *a, int s) {
        ll ans = 0;
        for (int i = 0; i < 6; i++, s >>= 1)
            if (s & 1)
                ans = (ans * P2 % MOD2 + a[i] % MOD2) % MOD2;
        return ans;
    }

    inline int calcHash3(int *a, int s) {
        ll ans = 0;
        for (int i = 0; i < 6; i++, s >>= 1)
            if (s & 1)
                ans = (ans * P3 % MOD3 + a[i] % MOD3) % MOD3;
        return ans;
    }

    inline int get(int *a, int s) {
        int hash1 = calcHash1(a, s);
        int hash2 = calcHash2(a, s);
        int hash3 = calcHash3(a, s);

        for (int tmp = point[hash1]; tmp; tmp = nxt[tmp])
            if (v1[tmp] == hash2 && v2[tmp] == hash3) 
                return cnt[tmp]++;

        tot++; 
        nxt[tot] = point[hash1]; point[hash1] = tot; cnt[tot] = 1;
        v1[tot] = hash2; v2[tot] = hash3;
        return 0;
    }   

};

ll f[MaxS]; 
int a[MaxN][6], cnt[MaxS];
int n, k;

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

int main() {
    n = getnum(); k = getnum();
    for (int i = 1; i <= n; i++)
        for (int j = 0; j < 6; j++)
            a[i][j] = getnum();
    
    for (int s = 0; s < MaxS; s++) {
        for (int tmp = s; tmp; tmp >>= 1)
            cnt[s] += (tmp & 1);
    }

    for (int s = 0; s < MaxS; s++) {
        if (cnt[s] < k) continue;

        hash::clear();
        for (int i = 1; i <= n; i++)
            f[s] += hash::get(a[i], s);
    }

    ll ans = 0;
    for (int s = 0; s < MaxS; s++) {
        if (cnt[s] != k) continue;
        for (int tmp = s; tmp < MaxS; tmp = (tmp + 1) | s) {
            if ((cnt[tmp] - cnt[s]) % 2)
                ans -= f[tmp];
            else
                ans += f[tmp];
        }
    }

    cout << ans << endl;

    return 0;
}
```