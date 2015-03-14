title: "[Solution][CF ABBY Cup 3.0(contest316) G3]Good Substrings"
date: 2015-01-26 21:10:51
tags: [CF,后缀自动机,鸟哥神题系列]
categories: 题解
---
深入理解自动机
<!--more-->
## 题目描述
[传送门](http://codeforces.com/contest/316/problem/G3)

## 分析
求子串出现的次数，子串还不重复，显然后缀自动机。

刚开始用了一种特别麻烦的方法，结果调试了半天还`TLE`了。后来看了`VFK`大爷的题解才恍然大悟。

将主字符串和底下限制条件的字符串都加入到一个自动机内。注意，每个字符串都从$Root$节点开始加。以前我们是通过$Right$集合的大小来维护子串出现的次数。但是显然因为有多个询问，我们可以开一个数组，维护针对每个限制条件里的字符串的$Right$集合大小。然后通过拓扑排序求出，同时检查拓扑排序当前的这个状态如果是主串上的状态，并且对每个限制都满足，则答案加上当前状态对应的主串种类数。

头一次见把多个字符串扔到一个自动机里，并且分开维护$Right$集合的题目。长见识了。

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <queue>
using namespace std;
const int MaxLen = 5e4 + 10;
const int MaxN   = 13;
struct SamNode {
    SamNode *pre, *ch[26];
    int maxLen, cnt[MaxN], deg;
    SamNode() {
        pre = 0; memset(ch, 0, sizeof(ch));
        maxLen = 0; memset(cnt, 0, sizeof(cnt));
        deg = 0;
    }
};
struct SamAM {
    SamNode pool[MaxN * MaxLen * 2], *root, *last;
    int tot;
    SamAM() {
        root = last = pool;
        tot = 0;
    }
    void insert(int c) {
        SamNode *p = last;
        SamNode *np = pool + (++tot); np->maxLen = p->maxLen + 1;
        while (p && !p->ch[c])
            p->ch[c] = np, p = p->pre;
        if (!p)
            np->pre = root;
        else {
            SamNode *q = p->ch[c];
            if (q->maxLen == p->maxLen + 1)
                np->pre = q;
            else {
                SamNode *nq = pool + (++tot);
                nq->maxLen = p->maxLen + 1;
                memcpy(nq->ch, q->ch, sizeof(q->ch));
                nq->pre = q->pre;
                q->pre = np->pre = nq;
                while (p && p->ch[c] == q)
                    p->ch[c] = nq, p = p->pre; 
            }
        } last = np;
    }
} sam;
queue<SamNode*> q;
char s[MaxLen];
int lim[MaxN][2];
int main() {
    scanf("%s", s); int len = strlen(s);
    for (int i = 0; i < len; i++)
        sam.insert(s[i] - 'a'), sam.last->cnt[0] = 1;
    int n; scanf("%d", &n);
    for (int i = 1; i <= n; i++) {
        scanf("%s%d%d", s, &lim[i][0], &lim[i][1]); 
        int len = strlen(s);
        sam.last = sam.root;
        for (int j = 0; j < len; j++)
            sam.insert(s[j] - 'a'), sam.last->cnt[i] = 1;
    }
    for (int i = 1; i <= sam.tot; i++)
        sam.pool[i].pre->deg++;
    for (int i = 0; i <= sam.tot; i++)
        if (!sam.pool[i].deg) q.push(sam.pool + i);
    int ans = 0;
    while (!q.empty()) {
        SamNode *now = q.front(); q.pop();
        if (!now->maxLen) continue;
        if (now->cnt[0]) {
            bool isok = true;
            for (int i = 1; i <= n; i++)
                if (!(lim[i][0] <= now->cnt[i] && now->cnt[i] <= lim[i][1])) {
                    isok = false; break;
                    }
            if (isok) ans += now->maxLen - now->pre->maxLen; 
        }
        if (now->pre) {
            for (int i = 0; i <= n; i++)
                now->pre->cnt[i] += now->cnt[i];
            now->pre->deg--;
            if (!now->pre->deg) q.push(now->pre);
        }
    } 
    printf("%d\n", ans);
}
```