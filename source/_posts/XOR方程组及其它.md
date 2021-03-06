title: "XOR方程组及其它"
date: 2014-12-28 17:26:12
tags: [高斯消元,xor,XOR方程组]
categories: 题解
---
对做过的关于`xor`的问题的总结。
<!--more-->

## 约定
假设所有的数字都为$2^{60}$以内

## XOR性质
满足交换律、结合律、且是自身的逆运算。每一位的`xor`都是独立的。二进制数比较大小从低到高比较。

## 两个数的XOR和问题

### 1.$N$个数中选出两个数，使得XOR和最大

对所有的数按照二进制建一棵`Trie`树，枚举一个数字，找出在Trie树上和这个数字对应的`XOR`和最大的数字。方法：尽量向与枚举数字该二进制位相反的方向在`Trie`树上前进。

$O(60N)$

### 2.$N$个点的边带权树，找一条XOR和最大的路径

令$F[i]$为点$i$到根的路径的`XOR`和。然后按照`例1`的方法处理。

题目：BZOJ1954（[题解](http://gaotianyu1350.gitcafe.io/2014/12/19/BZOJ1954-TheXorLongestPath/)）

### 3.给定一个序列，查询一个数$A$和区间$[l,r]$中哪个数的XOR和最大

可持久化`Trie`，方法类似主席树。在区间上查询的方法类似`例1`。

题目：BZOJ3261（[题解](http://gaotianyu1350.gitcafe.io/2014/12/20/BZOJ3261-%E6%9C%80%E5%A4%A7%E5%BC%82%E6%88%96%E5%92%8C/)）

### 4.给定一个序列，给定数字$A$，查询$A$和序列中某一数字的$K$大XOR和

维护`Trie`。用类似主席树查询的方法查询。

变形：查询前$K$大的序列中两数的XOR和。用堆维护每一个数字和其他数字的$K$大XOR和。如果从堆中提取出来，就把$K+1$大的结果再放回堆里。

题目：BZOJ3689（[题解](http://gaotianyu1350.gitcafe.io/2014/12/21/BZOJ3689-%E5%BC%82%E6%88%96%E4%B9%8B/)）

### 小结
遇到两个数的XOR问题，先固定一个数，然后用`Trie`解决

## 多个数的XOR和问题

### 5.从$N$个数字中选出若干个使得XOR和为$K$，给出方案或指出不可行。

求`线性基`。

什么是`线性基`？`线性基`类似向量中的`基底`。我们首先将所有的数字拆成二进制后构成一个矩阵。利用类似高斯消元的方法，求出一个`线性无关`组。`线性无关`说白了就是各项不成比例，也就是转成向量后不平行。这样求出来的一个矩阵再乘上一个系数（表示每一个数选还是不选）就能唯一不重复地表示所有可能的XOR和，这就是`线性基`。

一段高效简洁的求线性基的代码。复杂度$O(60N)$
```
inline void calcLinearBase() {
    for (int i = 1; i <= n; i++)
        for (int j = MAXBIT; j >= 0; j--)
            if (a[i] & (1 << j)) {
                if (!lb[j]) { lb[j] = a[i]; break; }
                else a[i] ^= lb[j];
            }
}
```
其中`lb[j]`中存储的是最高位为`j`的线性基中的元素。利用线性基，我们可以求指定`XOR`和的方案，求指定XOR是第几大，求第$K$大的XOR和，求不同XOR和的个数。总之用处多多。

就这个例题来说，求完`线性基`后，从高到低循环每个线性基，如果答案在线性基的最高位上是$1$，就把答案XOR上这个线性基。如果最后不为$0$，那么这个答案无法得到。

### 6.求最大XOR和

求完线性基后从高到低枚举，考虑用优还是不用更优即可。

### 7.XOR和种数

$2^k$，$k$为线性基中元素的个数。

### 8.求XOR和与$K$的XOR和最大

同`例6`。

### 9.求$K$大的XOR和

如果是只算所有不同的XOR和，那么可以把每个线性基近似当成二进制数上的一位。从高向低枚举每个二进制数，如果执行会让答案变大的操作，排名就要加相应的$2$的多少次方，否则不加。求解过程有点像倍增

如果算不同的XOR和，那么对于每个XOR和，不同的方案有$2^{n-k}$种，其中$n$为数字的总个数，而$k$为线性基中元素的个数。先把$K$处理成只算不同XOR和时候的排名，然后再按照上述方案求解。

### 10.求指定XOR和的排名

是`例9`的逆操作，不再赘述。

题目：BZOJ2844（[题解](http://gaotianyu1350.gitcafe.io/2014/12/19/BZOJ2844-albus/)）

### 小结
求选择若干个数XOR和的问题，先求解线性基，再利用线性基的性质求解。

## XOR和在图和树中的应用

### 10.无向图中的XOR和最大割

设$h\_i$为与$i$相邻的边的权值的XOR和。一个割的XOR和等于所有$S$集中的$i$的$h\_i$的XOR和，也等于所有$T$集中的$i$的$h\_i$的XOR和。问题转成`例6`。利用的是边的两点在同一个集中时，它的权值会相互抵消。

### 11.无向图中XOR最大环（允许环走重复的路）

首先看结论：*一个无向图中有且仅有$M-N+1$个独立回路。（独立回路是指不能由其它独立回路凑出来的回路）。*
证明方法详见莫涛的`PPT`。是从树的形态不断加边推导过来的。

这样的话我们可以先求出所有独立回路的XOR和。然后任意组合。问题转化为`例6`

问：如果所选的独立回路并不相交怎么办？ 不用担心，可以想象成从这个回路上蹦到另一个回路以后原路返回。

### 12.XOR最长路（允许重复走）

任意两条路径合起来都是一个环（允许重复走）。那么我们可以随便选择一条路径，再求出所有独立环，从独立环中选出若干个使得和这条路径的XOR和最大。问题转化为`例8`。

题目：BZOJ2115（[题解](http://gaotianyu1350.gitcafe.io/2014/12/20/BZOJ2115-Xor/)）
BZOJ2322（[题解](http://gaotianyu1350.gitcafe.io/2014/12/20/BZOJ2322-%E6%A2%A6%E6%83%B3%E5%B0%81%E5%8D%B0/)）**这题非常有意思一定要看！**

### 小结
选择若干个元素的XOR和问题，尽量转化为`例6`和`例8`的简单形式求解。

## 开关问题
给定$n$个节点，每个节点都有两个状态$0$和$1$。每个节点有一个影响集合$S$，表示反转这个节点状态的同时也会反转$S$集合中节点的状态。给定初始状态和目标状态，求方案|方案数|是否可行。

这一类问题的思路为列出XOR方程组。首先得到`改变状态`为目标状态XOR`初始状态`。设未知数${x\_0,x\_1,x\_2,...,x\_{n-1}}$分别表示第$1,2,3,...,n$项的状态。假设一号节点的最终状态为$B\_0$,$A\_{0,i}$表示第$i+1$个元素是否会影响第$1$个元素，则可以得到方程：
$$ A\_{0,0}x\_0+A\_{0,1}x\_1+...+A\_{0,n-1}x\_{n-1}=B\_0 $$

其它的$n-1$个方程也可以这样得出。这样我们就可以用高斯消元解这个`XOR`方程组了。关于方案数：

无解：$r(A)=r(\overline A)$，增广矩阵的秩和系数矩阵的秩不相等，也就是消元之后出现$(0,0,0,0,...,a)且a\neq 0$的情况。
有唯一解：$r(A)=n$，系数矩阵的秩等于$n$，也就是消元消除了完美的倒三角。
有无数解：$r(A)<n$，系数矩阵的秩小于$n$，也就是消元后三角形有残缺。对于xor方程组来说，解的个数为$2^{n-r(A)}$。

矩阵的秩就是矩阵中线性无关的行（或者纵列）的最大数目，通俗点说就是方程组中本质不相同（系数不成比例）的方程的数目。
增广矩阵指系数矩阵加常数矩阵。

消元的时候如果遇到某元的系数为$0$，不要退出，而是跳过，并累加一下个数。最后方案数为$2$的这个个数次方。还要扫一遍看看有没有哪一行系数都是$0$但常数不为$0$，那么就是无解。

在多解的情况下求所有未知数的和最大|最小？暴力枚举系数被约掉的项是$0$还是$1$，最优化剪枝。

题目：BZOJ1923（[题解](http://gaotianyu1350.gitcafe.io/2014/12/16/BZOJ1923-%E5%A4%96%E6%98%9F%E5%8D%83%E8%B6%B3%E8%99%AB/)）
POI1830（[题解](http://gaotianyu1350.gitcafe.io/2014/12/17/POJ1830-%E5%BC%80%E5%85%B3%E9%97%AE%E9%A2%98/)）
BZOJ2466（[题解](http://gaotianyu1350.gitcafe.io/2014/12/17/BZOJ2466-%E6%A0%91/)）

## 待填坑

莫涛`PPT`上的思考题。

## 引用与鸣谢

莫涛PPT：[高斯消元解XOR方程组](http://wenku.baidu.io/link?url=GEdOCrRk1KcIOvLdiVES8GhfbVOjnnIJkYbpLQyiSpm9BtxKjfLyV4-NXXPi8DRE3FS4jejTHDqy3n8uXTqy-UNSBxsCnWJn78gS10Zzl2e)
感谢[ZKY](http://blog.csdn.net/iamzky)神犇在`可持久化数据结构`和`线性基`方面提供的帮助。有几道题目也是参考了`ZKY`神犇的题解。