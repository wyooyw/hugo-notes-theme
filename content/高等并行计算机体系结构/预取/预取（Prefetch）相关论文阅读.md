---
linktitle: 预取（Prefetch）相关论文阅读
summary: Summary of 预取（Prefetch）相关论文阅读
type: book
---

## 1 Using a User-Level Memory Thread for Correlation Prefetching

### 基本概念

correlation table
processor-side prefetching
memory-side prefetching

## 2.Push vs. pull: data movement for linked data structures
基本概念：
push / pull model
linked data structures (LDS)


## 3. Design and evaluation of a compiler algorithm for prefetching

### polyhedron

对于大多数科学计算场景，都可以表示为嵌套循环，也称为loop nest。

举例：

矩阵乘法

```
C[ : , : ] = 0
for (i = 0; i < n; i++)
    for (j = 0; j < n j++)
        for (k = 0; k < n; k++)
            C[ i , j ] += A[ i , k ] + B[ k , j ]
```

LU分解

```
for I1 := 1 to n do
	for I2 := I1+1 to n do
		A[I2,I1] /= A[I1,I1]
		for I3 := I1+1 to n do
			A[I2,I3] -= A[I2,I1] * A[I1,I3]
```

在k层嵌套循环中，每个interation都对应一组循环下标（I1,I2,...,Ik），故可以将每个iteration看做一个k维空间中的向量。假设各分量是连续的，再加上循环边界的约束，则iteration的集合构成了一个**凸多面体**（1个约束对应一个半空间，所有约束相当于2k个半空间的交，是凸集；同时因为循环边界是线性的，所以该凸集的边界是线性的，所以是一个凸多面体；当循环边界都是常数时，则多面体退化为与坐标轴垂直的立方体）。

### reuse
reuse在这篇文章里分成了spatial reuse, temporal reuse 和group reuse。

对reuse更详细的分类和定义，见论文《A data locality optimizing algorithm》的4.1节“Types of reuse”：

> Reuse occurs when a reference within a loop accesses the same data location in different iterations. We call this **self-temporal reuse**. Likewise, if a reference accesses data on the same cache line in different iterations, it is said to possess **self-spatial reuse**. Furthermore, different references may access the same locations. We say that there is **group-temporal reuse** if the references refer to the same location, and **group-spatial reuse** if they refer to the same cache line.

根据定义，显然任何一个temporal reuse 是 spatial reuse

> By definition, temporal reuse is a subset of spatial reuse; reusing the same location is trivially reusing the same cache line.

举例：


### reuse vector space

#### 1) temporal reuse vector space

设loop nest的两次iteration下标为$i_1,i_2$，设某reference的下标为$Hi+c$，则存在temporal reuse当且仅当
$$
H i_1+c=Hi_2+c
$$
进而得到
$$
H(i_1-i_2)=0
$$
此时称$H$的核空间为**temporal reuse vector space**。



#### 2) spatial reuse vector space

本论文里没写，一句”Similar analysis can be used to find spatial reuse“带过了。

于是参照论文《A data locality optimizing algorithm》给出定义：

这里假设了张量的最后一维刚好能放到一个cacheline中

设loop nest的两次iteration下标为$i_1,i_2$，设某reference的下标为$Hi+c$，设$S$为$n*n$的单位向量将最后一个元素设为0，则存在spatial reuse当且仅当
$$
S(Hi_1+c)=S(Hi_2+c)
$$
化简得
$$
(SH)(i_1-i_2)=0
$$
令$H_S=(SH)$，称$H_S$的核空间为**spatial reuse vector space**



#### 3）uniformly generated

如果两个不同的reference的下标$H_1i_1+c1$，$H_2i_2+c_2$，差值是常数，则称这两个reference是uniformly generated的，它们之间存在group reuse。



### 未解决的问题

1）对polyhedren的定义，是否需要循环边界必须是仿射函数？

2）iteration如果不在最内层循环怎么办？例如矩阵乘法的初始化语句，以及很多场景中都有。

此时loopnest的定义是什么？我又怎么用论文中的方法来分析?

探讨Perfect loop nest 和Inperfect loop nest的文章：
《Blocking and array contraction across arbitrarily nested loops using affine partitioning》直接在imperfect loop nest上做操作

《Synthesizing transformations for locality enhancement of imperfectly-nested loop nests》和《Tiling imperfectly-nested loop nests.》将imperfect的转换成perfect的。

