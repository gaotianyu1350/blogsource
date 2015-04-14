title: "[Solution][BZOJ1879][SDOI2009]Bill的挑战"
date: 2015-04-01 17:23:51
tags: [BZOJ,SDOI,状压DP]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1879)

## 分析
 我真是弱到一定境界了……

刚开始以为这是什么暴搜+容斥……结果发现容斥根本没法做><后来才发现这就是个普通的状压DPQAQ

观察数据范围，发现非常小，很适合状压。令$f[i][j]$表示匹配到第$i$位，还有$j$表示的状态那些字符串匹配。初始为$f[0][2^n-1]=1$。令$trans[i][c]$表示如果第$i$位上字符为$c$的话，能够匹配的字符串。转移：
$$ f[i+1][s~ and~ trans[i+1][c]]+=f[i][s] $$

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
const int MaxN = 20;
const int MaxL = 60;
const int MaxS = (1 << 17);
const ll mod = 1e6 + 3;

int len, n, k;
int f[MaxL][MaxS], trans[MaxL][26];
char s[MaxN][MaxL];

inline void update(int &a, int b) {
    a += b;
    if (a >= mod) a -= mod;
}

int main() {
    int t;
    scanf("%d", &t);

    while (t--) {
        memset(f, 0, sizeof(f));
        memset(trans, 0, sizeof(trans));
        scanf("%d%d", &n, &k);
        for (int i = 1; i <= n; i++)
            scanf("%s", s[i]);
        len = strlen(s[1]);

        for (int i = 1; i <= n; i++) {
            int cur = 1 << (i - 1);
            for (int j = 0; j < len; j++)
                if (s[i][j] != '?')
                    trans[j][s[i][j] - 'a'] += cur;
                else {
                    for (int k = 0; k < 26; k++)
                        trans[j][k] += cur;
                }
        }

        int tarS = 1 << n;
        f[0][tarS - 1] = 1;
        for (int i = 0; i < len; i++)
            for (int s = 0; s < tarS; s++)
                if (f[i][s])
                    for (int k = 0; k < 26; k++)
                        update(f[i + 1][s & trans[i][k]], f[i][s]);

        int ans = 0;
        for (int s = 0; s < tarS; s++) {
            int cnt = 0;
            for (int tmp = s; tmp; tmp >>= 1)
                if (tmp & 1) cnt++;
            if (cnt == k)
                update(ans, f[len][s]);
        }

        printf("%d\n", ans);
    }
}
```