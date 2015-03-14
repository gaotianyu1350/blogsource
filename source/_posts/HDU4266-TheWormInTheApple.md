title: "[Solution][HDU4266]The Worm in the Apple"
date: 2015-02-03 20:42:53
tags: [HDU,三维凸包,三维几何]
categories: 题解
---
<!--more-->
## 题目描述
[传送门](http://acm.hdu.edu.cn/showproblem.php?pid=4266)

## 分析
三维凸包。

## 代码
```
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
#include <vector>
using namespace std;
const double inf = 1e20;
const double eps = 1e-8;
const int MaxN = 1e3 + 10;
inline int getnum() {
    int ans = 0; char c; bool flag = false;
    while ((c = getchar()) == ' ' || c == '\n' || c == '\r');
    if (c == '-') flag = true; else ans = c - '0';
    while ((c = getchar()) >= '0' && c <= '9') ans = ans * 10 + c - '0';
    return ans * (flag ? -1 : 1);
}
struct Point { 
    double x, y, z;
    Point(double xx = 0, double yy = 0, double zz = 0)
        : x(xx), y(yy), z(zz) {}
    double len() {
        return sqrt(x * x + y * y + z * z);
    }
    Point operator + (const Point o) const {
        return Point(x + o.x, y + o.y, z + o.z);
    }
    Point operator - (const Point o) const {
        return Point(x - o.x, y - o.y, z - o.z);
    }
    Point operator * (const double rate) const {
        return Point(x * rate, y * rate, z * rate);
    } 
    void Print() {
        printf("%.2f %.2f %.2f\n", x, y, z);
    }
};
inline int dcmp(double a, double b) {
    if (fabs(a - b) < eps) return 0;
    else return a > b ? 1 : -1;
}
inline double Dot(Point a, Point b) {
    return a.x * b.x + a.y * b.y + a.z * b.z;
}
inline Point Cross(Point a, Point b) {
    return Point(a.y * b.z - b.y * a.z, a.z * b.x - b.z * a.x, a.x * b.y - b.x * a.y);
}
struct Face {
    int v[3];
    Face(int a = 0, int b = 0, int c = 0) {
        v[0] = a; v[1] = b; v[2] = c;
    }
    Point Normal(Point *p) {
        return Cross(p[v[1]] - p[v[0]], p[v[2]] - p[v[0]]);
    }
    bool canSee(Point *p, Point q) {
        return dcmp(Dot(q - p[v[0]], Normal(p)), 0) > 0;
    } 
};
Point p[MaxN];
bool vis[MaxN][MaxN];
int n, cnt;
vector<Face> getCH() {
    vector<Face> cur;
    cur.push_back(Face(0, 1, 2));
    cur.push_back(Face(2, 1, 0));
    for (int i = 3; i < n; i++) {
        vector<Face> nxt;
        for (int j = 0; j < (int)cur.size(); j++) {
            Face f = cur[j];
            bool res = f.canSee(p, p[i]);
            if (!res) nxt.push_back(f);
            for (int k = 0; k < 3; k++)
                vis[f.v[k]][f.v[(k + 1) % 3]] = res;
        }
        for (int j = 0; j < (int)cur.size(); j++) {
            Face f = cur[j];
            for (int k = 0; k < 3; k++) {
                int a = f.v[k], b = f.v[(k + 1) % 3];
                if (vis[a][b] != vis[b][a] && vis[a][b])
                    nxt.push_back(Face(a, b, i)); 
            }
        }
        cur = nxt;
    }
    return cur;
}
inline double rand01() {
    return (double)rand() / RAND_MAX;
}
inline double randEps() {
    return (rand01() - 0.5) * eps; 
}
inline Point addNoise(Point p) {
    return Point(p.x + randEps(), p.y + randEps(), p.z + randEps());
}
inline double calcDis(Point *p, Point pp, Face f) {
    return fabs(Dot(pp - p[f.v[0]], f.Normal(p))) / f.Normal(p).len();
}
int main() {
    while (1) {
        scanf("%d", &n);
        if (!n) break;
        for (int i = 0; i < n; i++) {
            scanf("%lf%lf%lf", &p[i].x, &p[i].y, &p[i].z);
            p[i] = addNoise(p[i]); 
        }
        vector<Face> ch = getCH();
        scanf("%d", &cnt);
        while (cnt--) {
            Point q;
            scanf("%lf%lf%lf", &q.x, &q.y, &q.z);
            double ans = inf;
            for (int i = 0; i < (int)ch.size(); i++)
                ans = min(ans, calcDis(p, q, ch[i]));
            printf("%.4f\n", ans);
        }
    }
}
```