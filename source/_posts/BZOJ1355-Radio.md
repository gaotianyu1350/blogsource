title: "[Solution][BZOJ1355][Baltic2009]Radio Transmission"
date: 2015-01-21 10:42:58
tags: [BZOJ,KMP,扩展KMP]
categories: 题解
---
KMP or 扩展KMP？
<!--more-->
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=1355)

## 分析
如果用`扩展KMP`的话，求出`nxt`数组，找一个最小的$i$，使得$i+nxt_i=len$，这个$i$就是答案。

如果是`KMP`的话，枚举长度$i$，找到最后一个循环节结束的位置$pos$，然后看$fail_{pos}$是否等于$pos-i$，再看一下剩下的尾巴是否和开头匹配即可。

注意：无论是`扩展KMP`还是`KMP`都要注意看清数组的含义。如`KMP`中的$fail$是指的和当前位相等的位置，还是和当前位的前位相等。`扩展KMP`的$nxt$是指匹配位置还是长度。

以我的代码为例，我写的是`扩展KMP`，$nxt$指长度，有一句话
`if (i + nxt[i - wh] >= p || i > p)`
中的等号千万不能省去！！因为这个我挂了好几次……

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
using namespace std;
const int MAXN = 1e6 + 10;
char s[MAXN];
int nxt[MAXN], len;
inline int makeNxt() {
    nxt[0] = len;
    if (len == 1) return len;
    for (nxt[1] = 0; nxt[1] < len - 1 && s[nxt[1]] == s[nxt[1] + 1]; nxt[1]++);
    int wh = 1, p = nxt[1]; if (p == len - 1) return 1;
    for (int i = 2; i < len; i++) {
        if (i + nxt[i - wh] >= p || i > p) {
            wh = i; if (i > p) p = i;
            for (; p < len && s[p] == s[p - i]; p++);
            nxt[i] = p - i;
        } else nxt[i] = nxt[i - wh];
        if (i + nxt[i] == len) return i;
    }
    return len;
}
int main() {
    scanf("%d", &len);
    scanf("%s", s);
    int ans = makeNxt();
    printf("%d\n", ans);
}
```