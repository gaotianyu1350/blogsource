title: "[Solution][BZOJ1228][SDOI2009]E&D"
date: 2015-03-29 21:09:17
tags: [BZOJ,SDOI,博弈论,SG函数]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1228)

## 分析
因为每一组属于互不相干的子游戏，只要把它们的`SG函数`异或起来就能得到总的`SG函数`。所以现在的问题是如何求每一组的`SG函数`。打表以后就会发现有奇特的规律~

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int MaxN = 2e4 + 10;
const int MaxBit = 31;

ll cm[MaxBit + 2];

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

inline int calc(ll x, ll y) {
    for (int bit = 0; bit <= MaxBit; bit++) {
        ll size = cm[bit];
        ll cX = x / size + (x % size ? 1 : 0);
        ll cY = y / size + (y % size ? 1 : 0);

        if (cX % 2 == 0 || cY % 2 == 0) continue;

        bool isok = true;
        cX = x; cY = y;
        for (int j = bit; j >= 1; j--) {
            ll size = cm[j];
            cX = (cX - 1) % size + 1;
            cY = (cY - 1) % size + 1;
            if (cY < size - cX + 1) {
                isok = false;
                break;
            }
        }

        if (isok) return bit;
    }
    
    return -1;
}

int main() {
    cm[0] = 1;
    for (int i = 1; i <= MaxBit; i++)
        cm[i] = cm[i - 1] * 2;

    int t = getnum();
    while (t--) {
        int ans = 0, n = getnum() / 2;
        for (int i = 1; i <= n; i++) {
            int x = getnum(), y = getnum();
            int cur = calc(x, y);
            ans ^= cur;
        }
        if (ans) printf("YES\n");
        else printf("NO\n");
    }
}
```