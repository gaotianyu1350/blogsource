title: "[Solution][BZOJ3199][SDOI2013]escape"
date: 2015-03-25 20:17:54
tags: [BZOJ,SDOI,半平面交,计算几何]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3199)

## 分析
看到计算几何就头疼……

首先我们可以求出每个点控制的区域。边界是这个点与其它所有点的连线的中垂线再加上边界。半平面交即可。然后有公共边的两个人之见连边（公共点不算，因为公共点上至少有三个人能看见，而且走公共点不如绕开走），与边界相邻的向虚拟终点连边。然后用判断点是否在多边形内的方法找出起点，跑最短路即可。

据说有个$n=0$的数据点，特判一下~

<!--more-->
## 代码
```c++
#include <bits/stdc++.h>
using namespace std;

const double eps = 1e-10;
const double inf = 1e15;
const int MaxN = 700;
const int MaxM = MaxN * MaxN * 2;

inline int dcmp(const double &a, const double &b) {
    if (fabs(a - b) < eps) return 0;
    return a > b ? 1 : -1;
}

struct point {
    double x, y;
    point() {}
    point(double xx, double yy) : x(xx), y(yy) {}
    void verti() { swap(x, y); x = -x; }
    void rev() { x = -x; y = -y; }
    point operator + (const point &o) const { return point(x + o.x, y + o.y); }
    point operator - (const point &o) const { return point(x - o.x, y - o.y); }
    point operator * (const double &o) const { return point(x * o, y * o); }
    point operator / (const double &o) const { return point(x / o, y / o); }
    bool operator == (const point &o) const {
        return !dcmp(x, o.x) && !dcmp(y, o.y);
    }
};

inline double mtcross(const point &a, const point &b, const point &c) {
    return (b.x - a.x) * (c.y - b.y) - (c.x - b.x) * (b.y - a.y);
}

inline double cross(const point &a, const point &b) {
    return a.x * b.y - b.x * a.y;
}

struct line {
    point s, v;
    int idx;
    line() {}
    line(point ss, point vv) : s(ss), v(vv) {}
    line(point ss, point vv, int xx) : s(ss), v(vv), idx(xx) {}
    bool operator < (const line &o) const {
        double rate1 = atan2(v.y, v.x);
        double rate2 = atan2(o.v.y, o.v.x);
        return dcmp(rate1, rate2) < 0 || (!dcmp(rate1, rate2) && dcmp(mtcross(s, s + v, o.s), 0) < 0);
    }
};

inline point getInter(line a, line b) {
    point u = b.s - a.s;
    double rate = cross(u, b.v) / cross(a.v, b.v);
    return a.s + a.v * rate;
}

inline bool onRight(line a, point b) {
    return dcmp(mtcross(a.s, a.s + a.v, b), 0) <= 0;
}

inline bool inRange(point a, point b, point x) {
    return dcmp(min(a.x, b.x), x.x) <= 0 && dcmp(x.x, max(a.x, b.x)) <= 0 &&
           dcmp(min(a.y, b.y), x.y) <= 0 && dcmp(x.y, max(a.y, b.y)) <= 0;
}

point family[MaxN], limit, self;
int n;

struct block {
    bool ct[MaxN];
    line q[MaxN], l[MaxN];
    int tot, head, tail;

    inline void clear() {
        memset(ct, 0, sizeof(ct));
        tot = 0;
    }

    inline void init(int idx) {
        for (int i = 1; i <= n; i++)
            if (i != idx) {
                point mid = (family[i] + family[idx]) / 2;
                point u = family[i] - family[idx];
                u.verti();
                if (onRight(line(mid, u), family[idx]))
                    u.rev();
                l[++tot] = line(mid, u, i);
            }
    }

    inline void makeHalfInter() {
        l[++tot] = line(point(0, 0), point(inf, 0), n + 1);
        l[++tot] = line(point(limit.x, 0), point(0, inf), n + 1);
        l[++tot] = line(point(limit.x, limit.y), point(-inf, 0), n + 1);
        l[++tot] = line(point(0, limit.y), point(0, -inf), n + 1);
        sort(l + 1, l + 1 + tot);
        head = 0, tail = 0;
        for (int i = 1; i <= tot; i++) {
            if (i > 1 && !dcmp(cross(l[i - 1].v, l[i].v), 0))
                continue;
            while (head + 1 < tail && onRight(l[i], getInter(q[tail - 1], q[tail]))) tail--;
            while (head + 1 < tail && onRight(l[i], getInter(q[head + 1], q[head + 2]))) head++;
            q[++tail] = l[i];
        }
        while (head + 1 < tail && onRight(q[head + 1], getInter(q[tail - 1], q[tail]))) tail--;
        while (head + 1 < tail && onRight(q[tail], getInter(q[head + 1], q[head + 2]))) head++;

        for (int i = head + 1; i <= tail; i++) 
            ct[q[i].idx] = true;
    }

    inline bool inPolygon(point x) {
        int cnt = 0;
        line tmp = line(x, point(inf, 0));
        for (int i = head + 1; i <= tail; i++) {
            if (!dcmp(cross(tmp.v, q[i].v), 0)) continue;
            point inter = getInter(tmp, q[i]);
            point side1, side2;

            if (i == head + 1) 
                side1 = getInter(q[tail], q[i]);
            else
                side1 = getInter(q[i - 1], q[i]);
            if (i == tail)
                side2 = getInter(q[head + 1], q[i]);
            else
                side2 = getInter(q[i], q[i + 1]);
            if (dcmp(side1.y, side2.y) > 0) swap(side1, side2);

            if (inRange(side1, side2, inter) && inRange(tmp.s, tmp.s + tmp.v, inter)) {
                if (inter == side1) { 
                    cnt++;
                    continue;
                } else if (inter == side2) {
                    continue;
                } else {
                    cnt++;
                }
            }
        }
        return cnt % 2;
    }

} b[MaxN];

namespace spfa {
    int dis[MaxN];
    bool inq[MaxN];
    int point[MaxN], nxt[MaxM], v[MaxM], w[MaxM], tot;

    inline void addedge(int x, int y, int cost) {
        tot++; nxt[tot] = point[x]; point[x] = tot; v[tot] = y; w[tot] = cost;
        tot++; nxt[tot] = point[y]; point[y] = tot; v[tot] = x; w[tot] = cost;
    }

    inline void clear() {
        memset(point, 0, sizeof(point));
        memset(nxt, 0, sizeof(nxt));
        tot = 0;
    }

    inline int spfa(int s, int t) {
        memset(dis, 0x7f, sizeof(dis));
        memset(inq, 0, sizeof(inq));
        queue<int> q;
        q.push(s); dis[s] = 1; inq[s] = true;
        while (!q.empty()) {
            int now = q.front(); q.pop(); inq[now] = false;
            for (int tmp = point[now], u; u = v[tmp], tmp; tmp = nxt[tmp])
                if (dis[u] > dis[now] + w[tmp]) {
                    dis[u] = dis[now] + w[tmp];
                    if (!inq[u])
                        inq[u] = true, q.push(u);
                }
        }
        return dis[t];
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
    int t = getnum();
    while (t--) {
        n = getnum();
        int x, y; x = getnum(); y = getnum();
        limit = point(x, y);
        x = getnum(); y = getnum();
        self = point(x, y);
        for (int i = 1; i <= n; i++) {
            x = getnum(); y = getnum();
            family[i] = point(x, y);
        }

        if (!n)
            printf("0\n");
        else if (n == 1)
            printf("1\n");
        else {
            int Start = 0, Tar = n + 1;
            for (int i = 1; i <= n; i++) {
                b[i].clear();
                b[i].init(i);
                b[i].makeHalfInter();
                if (!Start && b[i].head + 2 < b[i].tail && b[i].inPolygon(self))
                    Start = i;
            }
            spfa::clear();
            for (int i = 1; i <= n; i++) {
                for (int j = i + 1; j <= n; j++) 
                    if (b[i].ct[j] && b[j].ct[i])
                        spfa::addedge(i, j, 1);
                if (b[i].ct[Tar])
                    spfa::addedge(i, Tar, 0);
            }
            printf("%d\n", spfa::spfa(Start, Tar));
        }
    }
}
```