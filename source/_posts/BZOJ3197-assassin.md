title: "[Solution][BZOJ3197][SDOI2013]assassin"
date: 2015-03-26 17:23:32
tags: [BZOJ,SDOI,树同构,Hash,KM,二分图,DP]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3197)

## 分析
题目的意思是找到一种树同构，然后问最少次数变换达到目标状态。

关于树同构：我们可以通过`Hash`的方法判断两棵子树是否同构。令$H_v$表示$v$为根的子树的`Hash`值。计算方法如下：
$$ H_v = ((((ap~xor~ H_1)p~xor~ H_2)p~xor H_3)...)b $$
其中$a,p,b$为常数，$H_1,H_2,...$为儿子的`Hash`值（排序后），公式中省略了取模（我懒得打了）。

注意：不能用平时求字符串的时候用的`Hash`方法，那样碰撞的几率很大。不过如果在前面加上深度好像也是可以的。

现在还有一个蛋疼的问题，这是一棵无根树。没关系，一棵树的中心只有一个，我们可以以中心为根。如果中心在边上，就建一个新的点。

令`F[i][j]`表示以$i$为根的子树对应以$j$为根的子树的情况下的最小代价。显然$i$和$j$必须深度相同且`Hash`值相同。然后通过对子树里面的节点建立二分图然后最大完美匹配即可。由于需要子树信息，所以`DP`的时候就按照深度第一关键字，`Hash`值第二关键字排序，按照顺序`DP`即可。

**手残毁一生！**排序的比较器里面因为打错了一个数字结果调试了一个小时……

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

typedef long long ll;
typedef vector<int>::iterator pVec;
const int MaxN = 720;
const int inf = 1e9;
const int Limit = 1e5;

bool disable[MaxN * 2];
int point[MaxN], nxt[MaxN * 2], v[MaxN * 2], fa[MaxN], deep[MaxN], tot;
int firstSt[MaxN], finalSt[MaxN];
int n;

inline void addedge(int x, int y) {
    tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y;
    tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x;
}

namespace km {
    int gra[MaxN][MaxN];
    bool luse[MaxN], ruse[MaxN];
    int l[MaxN], r[MaxN], result[MaxN];
    int slack[MaxN];

    inline void clear(int n) {
        for (int i = 1; i <= n; i++)
            for (int j = 1; j <= n; j++)
                gra[i][j] = -1;
        for (int i = 1; i <= n; i++)
            result[i] = 0;
    }

    bool find(int x, int n) {
        luse[x] = true;
        for (int i = 1; i <= n; i++)
            if (!ruse[i] && gra[x][i] != -1) {
                int tmp = gra[x][i] - (l[x] + r[i]);
                if (!tmp) {
                    ruse[i] = true;
                    if (!result[i] || find(result[i], n)) {
                        result[i] = x;
                        return true;
                    }
                } else
                    slack[i] = min(slack[i], tmp);
            } 
        return false;
    }

    inline int solve(int n) {
        if (!n) return 0;
        for (int i = 1; i <= n; i++)
            r[i] = 0;
        for (int i = 1; i <= n; i++) {
            l[i] = inf;
            for (int j = 1; j <= n; j++)
                if (gra[i][j] != -1)
                    l[i] = min(l[i], gra[i][j]);
        }

        for (int i = 1; i <= n; i++) {
            for(int j = 1; j <= n; j++) slack[j] = inf;
            int cnt = 0;
            while (1) {
                cnt++;
                if (cnt >= Limit) {
                    printf("Wrong!\n");
                    return -1;
                }
                for (int j = 1; j <= n; j++) luse[j] = ruse[j] = 0;
                if (find(i, n)) break;

                int minn = inf;
                for (int i = 1; i <= n; i++)
                    if (!ruse[i])
                        minn = min(minn, slack[i]);
                for (int i = 1; i <= n; i++) {
                    if (luse[i]) l[i] += minn;
                    if (ruse[i]) r[i] -= minn;
                    else slack[i] -= minn;
                }
            }
        }

        int ans = 0;
        for (int i = 1; i <= n; i++) ans += l[i] + r[i];
        return ans;
    }
}

namespace hash {
    const int MOD1 = 1060469;
    const int MOD2 = 1055897;
    const int A1 = 14221;
    const int A2 = 17737;
    const int B1 = 20707;
    const int B2 = 6577;
    const int P1 = 4481;
    const int P2 = 3187;

    int value1[MaxN], value2[MaxN];

    inline bool isSame(int x, int y) {
        return value1[x] == value1[y] && value2[x] == value2[y];
    }

    inline void calcHash(int x) {
        vector<int> vec;
        for (int tmp = point[x], u; u = v[tmp], tmp; tmp = nxt[tmp])
            if (!disable[tmp] && u != fa[x])
                vec.push_back(value1[u]);
        sort(vec.begin(), vec.end());
        
        ll v1 = A1;
        for (pVec p = vec.begin(); p != vec.end(); p++)
            v1 = (v1 * P1 % MOD1 ^ *p) % MOD1;
        v1 = v1 * B1 % MOD1;

        vec.clear();
        for (int tmp = point[x], u; u = v[tmp], tmp; tmp = nxt[tmp])
            if (!disable[tmp] && u != fa[x])
                vec.push_back(value2[u]);
        sort(vec.begin(), vec.end());

        ll v2 = A2;
        for (pVec p = vec.begin(); p != vec.end(); p++)
            v2 = (v2 * P2 % MOD2 ^ *p) % MOD2;
        v2 = v2 * B2 % MOD2;

        value1[x] = v1;
        value2[x] = v2;
    }
}

namespace fix {
    int dis[MaxN];
    bool vis[MaxN];

    inline int bfs() {
        queue<int> q;
        q.push(1); vis[1] = true;
        int awayP = 1;

        while (!q.empty()) {
            int now = q.front(); q.pop();
            if (dis[now] > dis[awayP]) awayP = now;
            for (int tmp = point[now], u; u = v[tmp], tmp; tmp = nxt[tmp])
                if (!vis[u])
                    vis[u] = true, dis[u] = dis[now] + 1, q.push(u);
        }

        memset(dis, 0, sizeof(dis));
        memset(vis, 0, sizeof(vis));
        q.push(awayP); vis[awayP] = true;
        int tar = awayP;

        while (!q.empty()) {
            int now = q.front(); q.pop();
            if (dis[now] > dis[tar]) tar = now;
            for (int tmp = point[now], u; u = v[tmp], tmp; tmp = nxt[tmp])
                if (!vis[u])
                    vis[u] = true, dis[u] = dis[now] + 1, fa[u] = now, q.push(u);
        }

        int d = dis[tar], r = dis[tar] / 2;
        int thetype = d % 2;
        int x, y, root = tar, last;

        while (dis[root] != r)
            last = root, root = fa[root];
        x = root, y = last;

        if (thetype) {
            n++;
            for (int tmp = point[x], u; u = v[tmp], tmp; tmp = nxt[tmp])
                if (u == y) {
                    disable[tmp] = true;
                    break;
                }
            for (int tmp = point[y], u; u = v[tmp], tmp; tmp = nxt[tmp])
                if (u == x) {
                    disable[tmp] = true;
                    break;
                }
            addedge(n, x);
            addedge(n, y);
            root = n;
        }

        fa[root] = 0;
        return root;
    }
}

namespace dp {
    int f[MaxN][MaxN];
    int pos[MaxN];

    bool cmp(const int &a, const int &b) {
        return deep[a] > deep[b] || (deep[a] == deep[b] && hash::value1[a] < hash::value1[b]);
    }

    void dfs(int now) {
        deep[now] = deep[fa[now]] + 1;
        for (int tmp = point[now], u; u = v[tmp], tmp; tmp = nxt[tmp])
            if (!disable[tmp] && u != fa[now]) {
                fa[u] = now;
                dfs(u);
            }
        hash::calcHash(now);
    }

    void dp() {
        memset(f, -1, sizeof(f));

        for (int i = 1; i <= n; i++)
            pos[i] = i;
        sort(pos + 1, pos + 1 + n, cmp);

        for (int st = 1; st <= n; st++) {
            int last = st;
            while (last <= n && deep[pos[last + 1]] == deep[pos[st]] && hash::isSame(pos[last + 1], pos[st])) 
                last++;
            for (int i = st; i <= last; i++)
                for (int j = st; j <= last; j++) {
                    int tot = 0, X = pos[i], Y = pos[j];
                    for (int tmp = point[X], u; u = v[tmp], tmp; tmp = nxt[tmp])
                        if (!disable[tmp] && u != fa[X]) 
                            tot++;
                    km::clear(tot);

                    int idx = 1, idy = 1;
                    for (int tmp = point[X], u; u = v[tmp], tmp; tmp = nxt[tmp])
                        if (!disable[tmp] && u != fa[X]) {
                            idy = 1;
                            for (int tmp2 = point[Y], u2; u2 = v[tmp2], tmp2; tmp2 = nxt[tmp2])
                                if (!disable[tmp2] && u2 != fa[Y]) {
                                    km::gra[idx][idy] = f[u][u2];
                                    idy++;
                                }
                            idx++;
                        }
                
                    if (idx != idy) {
                        printf("Wrong!\n");
                        exit(0);
                    }

                    f[X][Y] = km::solve(tot);
                    if (f[X][Y] == -1) {
                        printf("Wrong!\n");
                        return;
                    }
                    f[X][Y] += (firstSt[X] != finalSt[Y]) ? 1 : 0;
                }

            st = last;
        }
    }
}

inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}

int main() {
    n = getnum();
    for (int i = 1; i < n; i++) {
        int x = getnum(), y = getnum();
        addedge(x, y);
    }
    for (int i = 1; i <= n; i++) firstSt[i] = getnum();
    for (int i = 1; i <= n; i++) finalSt[i] = getnum();

    int root = fix::bfs();
    dp::dfs(root);
    dp::dp();
    printf("%d\n", dp::f[root][root]);

    return 0;
}
```