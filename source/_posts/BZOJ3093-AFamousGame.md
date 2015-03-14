title: "[Solution][BZOJ3093]A Famous Game"
date: 2015-03-10 18:51:47
tags: [贝叶斯公式,概率,全概率公式,组合数]
categories: 题解
---
## 题目描述
[传送门](http://www.lydsy.com/JudgeOnline/problem.php?id=3093)

## 分析
我们定义事件$A$为拿出$P$个球，其中有$Q$个为红色，事件$B$为再拿出一个球为红色，事件$N\_k$为原来$n$个球中一共有$k$个球。先不要着急推式子，我们先看看能算出什么。

首先$P(N\_k)=\frac{1}{n+1}$是题目中给出的，好说。

用组合数可以推出$P(A|N\_k)=\frac{\binom{k}{q}\binom{n-k}{p-q} }{\binom{n}{p} }$。

$P(B|A)$是我们要求的，先不推。
但是$P(B|AN\_k)$还是比较好推的，$P(B|AN\_k)=\frac{k-q}{n-p}$

下面来推答案$P(B|A)$：

$$
\begin{align}
P(B|A)=&\frac{P(AB)}{P(A)}\\\
=&\frac{\sum\_{k=0}^n P(B|AN\_k)P(A|N\_k)P(N\_k)}{\sum\_{k=0}^n P(A|N\_K)P(N\_k)} \\\
\end{align}
$$

其中(1)用了贝叶斯公式，(2)用了全概率公式。

现在我们把前面已经推出来的一些结论带进去，得到：

$$
P(B|A)=\frac{ \sum\_{k=0}^n\binom{k}{q}\binom{n-k}{p-q}(k-q) }{\sum\_{k=0}^n\binom{k}{q}\binom{n-k}{p-q}(n-p) } \\\
$$

发现式子里面有个讨厌的$k$，难道要枚举吗？不用不用，我们可以想办法让$k$消失，变成：

$$
P(B|A)=\frac{ \sum\_{k=0}^n\binom{k}{q+1}\binom{n-k}{p-q}(q+1) }{\sum\_{k=0}^n\binom{k}{q}\binom{n-k}{p-q}(n-p) } \\\
$$

对于组合数，我们可以用这个变形：
$$ \sum\_{k=0}^n \binom{k}{q}\binom{n-k}{p-q}=\binom{n+1}{p+1} $$

代入得到：

$$
\begin{align}
P(B|A)=&\frac{ \binom{n+1}{p+2}(q+1) }{ \binom{n+1}{p+1}(n-p) } \\\
=& \frac{q+1}{p+2}
\end{align}
$$

纳尼！答案这么简单？没错，这就是数学的神(sang)奇(xin)之(bing)处(kuang)……

得到的一些技巧：首先列出一些能够求出来的概率（特别是一些条件概率），然后用各种公式把要求的东西往上凑。最后再想办法消掉一些变量来提高求解的效率~还有就是看到超级简单的结果不要虚，说不定就是对的……

<!--more-->
## 代码
```c++
#include <iostream>
#include <cstdio>
using namespace std;
 
int main() {
    int n, p, q, t = 0;
    while (t++, scanf("%d%d%d", &n, &p, &q) != EOF) {
        printf("Case %d: %.4f\n", t, (double)(q + 1) / (p + 2));
    }
}
```