title: 多项式与FFT
date: 2014-11-20 19:06:44
tags: FFT
categories: 总结
---
终于学会FFT了……以下内容为算导搬运+实现代码，可能有许多写错的地方，请大家多多包涵……
<!--more-->
## 多项式
### 多项式的系数表达
$$ A(x)=\sum\_{j=0}^{n-1}{a\_jx^j} $$

其系数表达是一个由系数组成的向量$a=(a\_0,a\_1,...,a\_{n-1})$。
### 多项式值的计算
可利用*霍纳法则*（又名*秦九韶算法*）可在$O(n)$时间复杂度内完成
$$ A(x\_0)=a\_0+x\_0(a\_1+x\_0(a\_2+...+x\_0(a\_{n-2}+x\_0a\_{n-1})...)) $$

### 多项式的加法
对于多项式加法，如果$A(x)$和$B(x)$是次数界（*严格大于多项式中所有次数的整数*）为$n$的多项式，那么它们的和也是一个次数界为$n$的多项式$C(x)$。对所有属于定义域的$x$，都有$C(x)=A(x)+B(x)$。也就是说
若 
$$A(x)=\sum\_{j=0}^{n-1}a\_jx^j $$
$$ B(x)=\sum\_{j=0}^{n-1}b\_jx^j $$
则
$$C(x)=\sum\_{j=0}^{n-1}c\_jx^j $$
其中$c\_j=a\_j+b\_j$

可以$O(n)$计算出

### 多项式的乘法
对于多项式乘法，如果$A(x)$和$B(x)$分别是是次数界为$n\_a$和$n\_b$的多项式，那么它们的乘积是一个次数界为$n\_a+n\_b-1$，同时也是$n\_a+n\_b$的多项式$C(x)$。对于所有属于定义域的$x$，都有$C(x)=A(x)B(x)$。普通的计算方法是将$A(x)$中的每一项与$B(x)$中的每一项相乘，然后合并同类项。也可表示为
$$C(x)=\sum\_{j=0}^{2n-2}{c\_jx^j}$$

其中
$$ c\_j=\sum\_{k=0}^{j}a\_kb\_{j-k} $$

朴素算法$O(n^2)$
计算出的系统向量$c$也称为输入向量$a$和$b$的*卷积*，表示成$ c=a\otimes b $

### 多项式的点值表达
一个次数界为$n$的多项式$A(x)$的点值表达就是一个由$n$个点值对所组成的集合
$$ {(x\_0,y\_0),(x\_1,y\_1),...,(x\_{n-1},y\_{n-1})} $$

使得对$ k=0,1,...,n-1 $,所有$x\_k$各不相同

### 插值
求值计算的逆（从一个多项式的点值表达确定其系数表达形式）称为插值
> *定理 插值多项式的唯一性* 对于任意$n$个点值对组成的集合，其中所有的$x$都不同，那么存在唯一的次数界为$n$的对应的多项式$A(x)$

基于上述定理，我们可以通过求解线性方程组的方法，在$O(n^3)$时间内完成插值

### 拉格朗日插值公式
*分析* 当$n=2$时，有点$(x\_0,y\_0),(x\_1,y\_1)$满足次数界为$2$的多项式$A(x)$。则
$$ A(x)=y\_0+\frac{y\_1-y\_0}{x\_1-x\_0}(x-x\_0) $$

化简可得

$$ A(x)=\frac{x-x\_1}{x\_0-x\_1}y\_0+\frac{x-x\_0}{x\_1-x\_0}y\_1 $$

其中$l\_0(x)=\frac{x-x\_1}{x\_0-x\_1},l\_1(x)=\frac{x-x\_0}{x\_1-x\_0}$叫做拉氏基函数，$A(x)$又可以表示为
$$ \sum\_{i=0}^{1}{l\_i(x)y\_i} $$

推广到次数界为$n$的情况，可得拉格朗日插值公式
$$ A(x)=\sum\_{k=0}^{n-1}{l\_k(x)y\_k} $$

也就是

$$ A(x)=\sum\_{k=0}^{n-1}{y\_k \frac{\prod\_{j\neq k}{(x-x\_j)}}{\prod\_{j\neq k}{(x\_k-x\_j)}}} $$

它可以使得$l_i(x_j)=\begin{cases}1, & i=j\\\0, & i\neq j\end{cases} $
所以求得的$A(x)$能使得所有$A(x\_i)=y\_i$
可以通过$O(n^2)$的时间求出系数

### 通过系数表示计算多项式加法和乘法
很容易证明复杂度是$O(n)$的，但是多项式乘法的时候需要注意，由于所求得的多项式的次数界是$2n$，所以输入的多项式$A(x),B(x)$必须要扩展点值表达到$2n$个点

## 快速傅里叶变换 FFT
*目的* 加速系数表达的多项式乘法到$O(nlogn)$
### 思路与流程
*1.加倍次数界* 加入$n$个系数为$0$的高阶系数，将多项式$A(x)$和$B(x)$变为次数界为$2n$的多项式
*2.求值* 应用$2n$阶的FFT计算$A(x)$和$B(x)$的点值表达。复杂度$O(nlogn)$
*3.逐点相乘* 通过逐点相乘计算$C(x)$的点值表达。复杂度$O(n)$
*4.插值* 应用FFT构造出多项式$C(x)$的系数表达。复杂度$O(nlogn)$
### 单位复数根
*$n$次单位复数根*是满足$\omega ^n=1$的复数$\omega$。$n$次单位复数根恰好有$n$个：对于$k=0,1,...,n-1$，这些根是$e^{2\pi ik/n}$
*解释* 根据欧拉公式可知，$e^{2\pi i}=1$，所以$\omega\_n=e^{2\pi i/n}$是一个$n$次单位复数根，称为*主$n$次单位根*，所以的其他$n$次单位根都是$\omega\_n$的幂次。
$n$个$n$次单位复数根在乘法意义下形成了一个群，因为$\omega\_n^n=\omega\_n^0=1$，$\omega\_n^i\omega\_n^j=\omega\_n^{i+j}=\omega\_n^{(i+j)mod~n}$，$\omega\_n^{-1}=\omega\_n^{n-1}$(可通过共轭复数的性质推导)。下面给出一些引理
> *消去引理* 对任何整数$n\geqslant 0$，$k\geqslant 0$，以及$d\geqslant 0$，
$$ \omega\_{dn}^{dk}=\omega\_n^k $$

*证明* 
$$ \omega\_{dn}^{dk}=(e^{2\pi i/dn})^{dk}=(e^{2\pi i/n})^k=\omega\_n^k $$

> *推论* 对任意偶数$n>0$，有
$$ \omega\_n^{n/2}=\omega\_2=-1 $$
> *折半引理* 如果$n>0$为偶数，那么$n$个$n$次单位复数根的平方的集合就是$n/2$个$n/2$次单位复数根的集合

*证明* 根据*消去引理*，对任意非负整数$k$，我们有$(\omega\_n^k)^2=\omega\_{n/2}^k$。注意，如果对所有的$n$次单位复数根进行平方，那么获得每个$n/2$次单位根正好两次，因为
$$ (\omega\_n^{k+n/2})^2=\omega\_n^{2k+n}=\omega\_n^{2k}\omega\_n^n=\omega\_n^{2k}=(\omega\_n^k)^2 $$

折半引理对于用分治策略来对多项式的系数表达和点值表达进行相互转换时非常重要的，因为它保证递归子问题的规模只是递归调用的一半
> *求和引理* 对任意整数$n\geqslant 1$和不能被$n$整除的非负整数$k$，有
$$ \sum\_{j=0}^{n-1}{(\omega\_n^k)^j}=0 $$

*证明* 利用等差数列的前$n$项和公式
$$ \sum\_{j=0}^{n-1}{(\omega\_n^k)^j}=\frac{(\omega\_n^k)^n-1}{\omega\_n^k-1}=\frac{(\omega\_n^n)^k-1}{\omega\_n^k-1}=\frac{(1)^k-1}{\omega\_n^k-1}=0 $$

因为$k$不能被$n$整除，所以可以保证分母不为$0$

### DFT
给定次数界为$n$的多项式$A(x)$的系数形式，求出它在$\omega\_n^0,\omega\_n^1,...,\omega\_n^{n-1}$的值$y\_0,y\_1,...y\_{n-1}$。向量$y=(y\_0,y\_1,...y\_{n-1})$就是系数向量$a=(a\_0,a\_1,...a\_{n-1})$的*离散傅里叶变换*，记做$y=DFT\_n(a)$

### FFT计算单位复数根处多项式的值
通过这种叫做*快速傅里叶变换*的方法，利用复数单位根的特殊性质，可以在$O(nlogn)$的时间内计算出$DFT\_n(a)$，而直接计算的方法所需的时间为$O(n^2)$。以下我们假设$n$为$2$的整数幂。
*FFT算法*利用了分治策略。我们定义两个新的次数界为$n/2$的多项式$A^{[0]}(x)$和$A^{[1]}(x)$：

$$ A^{[0]}(x)=a\_0+a\_2x+a\_4x^2+...+a\_{n-2}x^{n/2-1} $$
$$ A^{[1]}(x)=a\_1+a\_3x+a\_5x^2+...+a\_{n-2}x^{n/2-1} $$

可以注意到，$A^{[0]}(x)$包括$A(x)$中所有下标为偶数的系数，$A^{[1]}(x)$包含了所有下标为奇数的系数。那么有
$$ A(x)=A^{[0]}(x^2)+xA^{[1]}(x^2) $$

所以，求$A(x)$在$\omega\_n^0,\omega\_n^1,\omega\_n^2,...,\omega\_n^{n-1}$处的值的问题转换为：
1.求次数界为$n/2$的多项式$A^{[0]}(x)$和$A^{[1]}(x)$在点$(\omega\_n^0)^2,(\omega\_n^1)^2,...,(\omega\_n^{n-1})^2$的值
2.利用$ A(x)=A^{[0]}(x^2)+xA^{[1]}(x^2) $综合上述结果
根据*折半引理*，$(\omega\_n^0)^2,(\omega\_n^1)^2,...,(\omega\_n^{n-1})^2$并不是由$n$个不同的值组成，而是仅由$n/2$个$n/2$次单位复数根组成，每个根正好出现两次。因此，我们可以递归地对次数界为$n/2$的多项式$A^{[0]}(x)$和$A^{[1]}(x)$在$n/2$个$n/2$次单位复数根处进行求值。这些子问题与原始问题形式相同，但规模变为了原来的一半。
以下是递归程序的伪代码
`RECURSIVE-FFT(a)`
```
n=a.length
if n==1
    return a
wn=e^(2*pi*i/n)
w=1
a0=(a_0,a_2,...,a_n-2)
a1=(a_1,a_3,...,a_n-1)
y0=RECURSIVE-FFT(a0)
y1=RECURSIVE-FFT(a1)
for k=0 to n/2-1
    t=w*y1[k]
    y[k]=y0[k]+t
    y[k+n/k]=y0[k]-t
    w=w*wn
```
### 逆DFT
我们定义$DFT$的逆运算$a=DFT\_n^{-1}(y)$，表示通过点值表达求出系数表达，即*插值*

### 逆FFT在单位复数根处插值
我们可以得到等式
$$ y=V\_na $$

其中$V\_n$是一个由$\omega\_n$适当幂次填充成的$n$行$n$列的矩阵。
对$j,k=0,1,...,n-1$，$V\_n$的$(k,j)$处元素为$\omega\_n^{kj}$。对于$DFT$逆运算$a=DFT\_n^{-1}(y)$，可以通过将$y$乘以$V\_n$的逆矩阵$V\_n^{-1}$来进行处理。
> *定理* 对$j,k=0,1,...,n-1$，$V\_n^{-1}$的$(j,k)$处元素为$\omega\_n^{-kj}/n$

*证明* 
$$[V\_n^{-1}V\_n]\_{jj'}=\sum\_{k=0}^{n-1}(\omega\_n^{-kj}/n)(\omega\_n^{kj'})=\sum\_{k=0}^{n-1}{\omega\_n^{k(j'-j)}/n}$$
如果$j=j'$，则此和为$1$；否则，根据*求和引理*，此和为$0$。
据此，可以推导出$DFT\_N^{-1}(y)$
$$ a\_j=\frac{1}{n}=\sum\_{k=0}^{n-1}{y\_k\omega\_n^{-kj}} $$
我们会发现这和计算$DFT$的式子十分相似，我们只需要把$a$和$y$互换，用$\omega\_n^{-1}$替换$\omega\_n$，并将计算结果的每个元素除以$n$，这样我们也可以再$O(nlogn)$的时间内计算出$DFT\_n^{-1}$
通过*FFT*和*逆FFT*，可以再$O(nlogn)$的时间内把次数界为$n$的多项式在其系数表达和点值表达之间进行相互转换。
> *卷积定理* 对任意两个长度为$n$的向量$a$和$b$，其中$n$是$2$的次幂，
$$ a\otimes b=DFT\_{2n}^{-1}(DFT\_{2n}(a)·DFT\_{2n}(b)) $$
其中向量$a$和$b$用$0$填充，使其长度达到$2n$。并用“·”表示$2$个$2n$个元素组成的向量的点乘。

### 高效实现FFT
*蝴蝶操作*
在FFT递归实现中，for循环中包含了$t=\omega y\_k^{[1]}$，然后从$y\_k^{[0]}$中加上或者减去$t$，这一系列操作叫做蝴蝶操作。
*通过迭代方式来实现FFT*
假设我们初始调用FFT的时候有$8$个元素，递归结构如下：
$$(a\_0,a\_1,a\_2,a\_3,a\_4,a\_5,a\_6,a\_7)$$
$$(a\_0,a\_2,a\_4,a\_6)(a\_1,a\_3,a\_5,a\_7)$$
$$(a\_0,a\_4)(a\_2,a\_6)(a\_1,a\_5)(a\_3,a\_7)$$
$$(a\_0)(a\_4)(a\_2)(a\_6)(a\_1)(a\_5)(a\_3)(a\_7)$$
观察这颗递归树，可以发现如果我们按照最后一层元素的顺序排好序的话，就可以自下而上的进行求解。用`rev[i]`存储第`i`个元素在排序后应在的位置。可以发现`rev[i]`实际上就是`i`的二进制反转。比如上面的例子中，元素$a\_6$的位置为$3$，$6$的二进制表示为$110$，而$3$的二进制表示为$011$。可以通过$O(nlogn)$的预处理确定`rev[i]`。
假设初始时输入的系数存于`A[0..n-1]`中，先将系数按照`rev[i]`的顺序排好。然后先由$1$到$logn$循环（假设循环变量为$s$），表示当前进行到递归树中的哪一层，然后再循环$\frac{n}{2^s}$个起点，剩下的操作和前面的递归算法就一样了——合并我们上一层求出的结果。求出的结果仍然存在`A`数组中，以便下一层迭代的时候进行合并。
具体的代码实现并不长，如下
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
using namespace std;
const int  MAXN = 2e6 + 10;
const double pi = acos(-1);
struct complex
{
    double r, i;
    complex() { r = i = 0; }
    complex(double a, double b) : r(a), i(b) {}
    complex operator + (complex other) 
    { 
        return complex(r + other.r, i + other.i); 
    }
    complex operator - (complex other)
    {
        return complex(r - other.r, i - other.i);
    }
    complex operator * (complex other)
    {
        return complex(r * other.r - i * other.i, r * other.i + i * other.r);
    }
}a1[MAXN], a2[MAXN], c[MAXN], A[MAXN];
int tot, n, L, rev[MAXN] = {0};
inline void makerev()
{
    for (int i = 0; i < n; i++)
    for (int j = 0; j < L; j++)
        rev[i] = (rev[i] << 1) + ((i >> j) & 1);
}
inline void FFT(complex *a, int flag)
{
    for (int i = 0; i < n; i++) A[i] = a[rev[i]];
    memcpy(a, A, sizeof(complex) * n);
    for (int l = 1; l <= L; l++)
    {
        int m = 1 << l;
        complex wn = complex(cos(2 * pi / m), flag * sin(2 * pi / m));
        for (int i = 0; i < n; i += m)
        {
            complex w = complex(1, 0);
            for (int k = 0; k < m / 2; k++)
            {
                complex t = w * a[i + k + m / 2];
                complex u = a[i + k];
                a[i + k] = u + t;
                a[i + k + m / 2] = u - t;
                w = w * wn;
            }
        }
    }
    if (flag == -1) for (int i = 0; i < n; i++) a[i].r /= n;
}
int main()
{
    scanf("%d", &tot);
    for (int i = 0; i < tot; i++) scanf("%lf", &a1[i].r);
    for (int i = 0; i < tot; i++) scanf("%lf", &a2[i].r);
    for (L = 0, n = 1; n < tot * 2; L++, n <<= 1);
    makerev();
    FFT(a1, 1);
    FFT(a2, 1);
    for (int i = 0; i < n; i++) c[i] = a1[i] * a2[i];
    FFT(c, -1);
    for (int i = 0; i < tot * 2 - 1; i++)
        printf("%.6f ", c[i].r);
}
```
有几个需要注意的细节：
1.实际编程的时候，我们需要写一个复数类，重载复数运算，而且注意要用`double`。
2.`FFT`和`逆FFT`可以写成一个过程，通过传一个`flag`表示是`FFT`还是`逆FFT`
3.`逆FFT`最后结果除以$n$的时候虚数部分不需要除，因为最后的结果一定是实数。
4.一些边界条件很容易写错，要注意。
下面是一段`FFT`实现高精度乘法的程序
```c++
#include <cstdio>
#include <cstring>
#include <cmath>
#include <cstdlib>
#include <iostream>
#include <algorithm>
using namespace std;
const int  MAXN = 2e6 + 10;
const double pi = acos(-1);
struct complex
{
    double r, i;
    complex() { r = i = 0; }
    complex(double a, double b) : r(a), i(b) {}
    complex operator + (complex other) 
    { 
        return complex(r + other.r, i + other.i); 
    }
    complex operator - (complex other)
    {
        return complex(r - other.r, i - other.i);
    }
    complex operator * (complex other)
    {
        return complex(r * other.r - i * other.i, r * other.i + i * other.r);
    }
}a1[MAXN], a2[MAXN], c[MAXN], A[MAXN];
int tot, n, L, rev[MAXN] = {0}, sum[MAXN] = {0};
char s1[MAXN], s2[MAXN];
inline void makerev()
{
    for (int i = 0; i < n; i++)
    for (int j = 0; j < L; j++)
        rev[i] = (rev[i] << 1) + ((i >> j) & 1);
}
inline void FFT(complex *a, int flag)
{
    for (int i = 0; i < n; i++) A[i] = a[rev[i]];
    memcpy(a, A, sizeof(complex) * n);
    for (int l = 1; l <= L; l++)
    {
        int m = 1 << l;
        complex wn = complex(cos(2 * pi / m), flag * sin(2 * pi / m));
        for (int i = 0; i < n; i += m)
        {
            complex w = complex(1, 0);
            for (int k = 0; k < m / 2; k++)
            {
                complex t = w * a[i + k + m / 2];
                complex u = a[i + k];
                a[i + k] = u + t;
                a[i + k + m / 2] = u - t;
                w = w * wn;
            }
        }
    }
    if (flag == -1) for (int i = 0; i < n; i++) a[i].r /= n;
}
int main()
{
    scanf("%s%s", s1, s2);
    int len1 = strlen(s1), len2 = strlen(s2);
    if ((len1 == 1 && s1[0] == '0') || (len2 == 1 && s2[0] == '0'))
    {
        printf("0\n"); 
        return 0;
    }
    tot = max(len1, len2);
    for (int i = len1 - 1; i >= 0; i--) a1[len1 - i - 1] = complex(s1[i] - '0', 0);
    for (int i = len2 - 1; i >= 0; i--) a2[len2 - i - 1] = complex(s2[i] - '0', 0);
    for (L = 0, n = 1; n < tot * 2; L++, n <<= 1);
    makerev();
    FFT(a1, 1);
    FFT(a2, 1);
    for (int i = 0; i < n; i++) c[i] = a1[i] * a2[i];
    FFT(c, -1);
    for (int i = 0; i < n; i++) sum[i] = (int)(c[i].r + 0.5);
    for (int i = 0; i < n; i++)
    {
        sum[i + 1] += sum[i] / 10;
        sum[i] %= 10;
    }
    int len = len1 + len2 - 1;
    if (sum[len]) len++;
    for (int i = len - 1; i >= 0; i--) printf("%d", sum[i]);
    putchar('\n');
}
```
##引用和鸣谢
本文的大部分内容来源于*《算法导论》第三版*
感谢*ZKY*神犇的帮助，还有他的[`FFT`启蒙教程](http://blog.csdn.net/iamzky/article/details/22712347)
