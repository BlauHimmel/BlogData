---
title: "Guide to Computational Geometry Processing Foundations 阅读笔记(2) 样条线(Splines)"
date: 2018-03-03T19:30:20+08:00
draft: true
tags: ["Guide to Computational Geometry Processing Foundations", "阅读笔记"]
---

这一章会讨论在CAD中经常使用的B-Splines, NURBS(Non-Uniform Rational B-Splines). 枯燥的公式很多, 并且配图很少, 不太好去理解.

<!--more-->

B-Splines(以及其它的参数曲面)相较于细分曲面(Subdivision Surface)的主要缺点是: 曲面的每一个小块(Patch)之间的连续性在进行网格变形(Deformation)时很难维护, 因此细分曲面更适合做动画. 但是细分曲面在小块(Patch)之间的缝隙(Seam)处会出现问题.

# 参数化(Parameterization)

通常一个曲面是不能够被用单一地参数化的, 而是需要将原有的曲面分成若干个小的曲面(Patch). 这些小的曲面(Patch)通常只在边界处重合.

$\mathbf{x}\_{1}(U_1) \cap \mathbf{x}\_{2}(U_2) \in  \mathbf{x}\_{1}(\partial U_1) \cap \mathbf{x}\_{2}(\partial U_2)$

在微分几何中, 单独的参数化通常需要被定义为一个开集(Open Set). 曲面上的连续性条件$C^1$, $C^2$等可以由每一个小曲面(Patch)上独立的参数化构造出来. 每一个小曲面(Patch)上的参数化的连续性并不能保证整个曲面也具有相同的连续性. 为了保证整个曲面的连续性条件, 需要满足一些特定的边界条件.

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note2-splines/img-0.PNG" title = "">}}

# 基本函数与控制点(Basis Functions and Control Points)

对于上图给定的参数化$\mathbf{x}:U \rightarrow R^3$, 可以写为如下的形式

$\mathbf{x}(u,v) = \sum\_{l=1}^{n} \mathbf{c}\_{l} F_{l}(u,v)$

其中函数$F\_{l}:U \rightarrow R$ 被称为是基本函数(Basis Functions), $\mathbf{c}_{l} \in R^3$被称作是控制点(Control Points). 参数化$\mathbf{x}$的定义是独立于坐标系统, 当且仅当基本函数满足

$\sum_{l=1}^{n} F\_{l}(u,v) = 1, \forall (u,v) in U$

且基本函数$F_l$通常是单变量多项式(Univariate Polynomial), 有理函数(Rational Functions), 分段多项式(Piecewise Polynomial)或者分段有理函数(Piecewise Rational Function)的乘积, 即$F\_l (u,v) = G\_{l}(u) H\_{l}(v)$.

# 节点与线上的样条空间(Knots and Spline Spaces on the Line)

设函数$f:[a,b] \rightarrow R$是一个最高次幂为$d$的分段多项式(Piecewise Polynomial). 也就是说函数$f$上存在一系列的断点, 称之为节点(Knot). 每一段多项式都满足不同程度的可微性.

一个节点序列(Knot Sequence)或者节点向量(Knot Vector)是一个非递减的序列(Non-Decreasing Sequence), 记作$\mathbf{u}$

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note2-splines/img-1.PNG" title = "">}}

其中$a=u\_{d+1}$且$b=u\_{n+d+1}$. 如果$u\_{l} < u\_{l+1}=\dots= u\_{l+\nu-1}<u\_{l+\nu}$, 那么节点$u\_{l+1}=\dots= u\_{l+\nu-1}$的重数(Multiplicity)为$\nu$.

如果重数(Multiplicity)$\nu = 1$, 那么我们说节点是单纯(Simple)节点, 如果$\nu = d$则节点是全复(Full Multiplicity)节点. 需要注意, 对于B样条曲线的节点向量, 通常边界上的节点的重数为$d + 1$, 也就是说$u\_{1}=\dots=u\_{d+1}=a$, $u\_{n + d + 1}=\dots=u\_{n+2d+1}=a$(在介绍B样条曲线的时候会说明其原因).

举个例子, 对于序列$0,0,0,1,2,2,2,3,7,7,9,9,9$节点值0的重数值为三, 节点值1的重数值为一, 节点值2的重数为三, 节点值7的重数值为二, 节点值9的重数值为三. 如果在节点列表中是以全复节点开始, 接下来是单纯节点, 再以全复节点结束, 而且节点值为等差, 称为均匀(Uniform).

节点向量$\mathbf{u}$上最高次幂为$d$的样条空间(Spline Space)是$S\_{\mathbf{u}}^{d}=S_{\mathbf{u}}^{d}([a,b])=f:[a,b] \rightarrow R$, 且多项式$f|\_{u\_l, u\_{l+1}}$的最高次幂为$d$, $f$在重数(Multiplicity)为$\nu$的节点上是$C^{d-\nu}$连续的.

对于上面给出的节点向量$u$, 最多有$n$个节点区间(Knot Interval),$[u\_{d+1},u\_{d+2}]$, ..., $[u\_{d+n},u\_{d+n+1}]$, 且样条空间(Spline Space)中的每一个样条由$n$个多项式组成. 如果所有的节点都是单纯的, 且有n个多项式，则每次节点的重数(Multiplicity)增加1, 则多项式的个数减少1.

如果一个节点是单纯的, 则有两个多项式是$C^{d-1}$可微的(Differentiability), 如果一个节点是全复节点的, 则有两个多项式仅是连续的(Continuity), 如果重数比最高次幂$d$高, 则有两个多项式是不连续的(Discontinuously).

总结一下, 节点向量应该满足: 是非递减的, 数子重复的次数不能超过$d$.

# B样条(B-Splines)

B样条全称为Basis Splines, 它组成了样条空间上的一组基底(Basis), 且基底的支集最小(Minimal Support).

PS. 支集即由满足$f(x) \neq 0$的$x$构成的集合.

节点向量$\mathbf{u}$上最高次幂为$d$的B样条可以递归地定义为

$N^{0}\_{\mathbf{u},l}(u)=
\begin{cases}
1,u\_l \leqslant u < u_{l+1}\\\\\
0,otherwise
\end{cases}$

$N^{d}\_{\mathbf{u},l}(u)=\frac{u-u\_{l-1}}{u\_{l+d-1}-u\_{l-1}}N^{d-1}\_{\mathbf{u},l}(u)+\frac{u\_{l+d}-u}{u\_{l+d}-u\_{l}}N^{d-1}\_{\mathbf{u},l+1}(u)$

如果节点向量为前面一节中给出的节点向量, 则展开后有$n+d$个函数, 不过其中有一部分函数始终为0. 如果一个节点的重数$\nu > d$, 那么$d - \nu$个函数始终为0.

B样条满足下列条件或者性质:

1. B样条$N^{d}\_{\mathbf{u},l}(u)$在开区间$\[u\_{l},u\_{l+1}\]$上的限制条件为其多项式的最高次幂为$d$.

2. B样条$N^{d}\_{\mathbf{u},l}(u)$在重数为$\nu$的节点上满足$C^{d-\nu}$连续.

3. B样条$N^{d}\_{\mathbf{u},l}(u)$的支集(Support)为$\[u\_{l},u\_{l+d+1}\]$, 即在这个区间上的值不为0.

4. B样条$N^{d}\_{\mathbf{u},l}(u)$的集合(Collection)组成了样条空间上的一组基底, 且基的支集最小.

5. B样条构成了一组单位划分(Partition of Unity), 即$\sum_{l}N^{d}\_{\mathbf{u},l}(u)=1$.

6. 如果节点序列是一个整数集$Z$, 在多项式最高次幂$d$固定的情况向, B样条之间可以通过相互平移得到, 即$N^{d}\_{Z,l}(u)=N^{d}\_{Z,l}(u+1-l)$, 且满足细分方程(Refinement Equation)$N^{d}\_{Z,0}(u)=\sum_{l}N^{d}\_{Z,0}(2u-l)$

节点向量$u$上的B样条曲线的方程为

$\mathbf{r}(u)=\sum_{l=1}^{n+d}\mathbf{c}\_{l}N^{d}\_{\mathbf{u},l}(u)$

其中$\mathbf{c}\_{l}$为控制点(Control Points), 通常控制点并不在曲线上. 但是如果节点$u\_{l+1}=\dots=u\_{l+d}$是全复节点, 则$\mathbf{c}\_{l}=\mathbf{r}(u\_{l+d})$

控制点$\mathbf{c}\_{l}$只会影响曲线在区间$\[u\_{l}, u\_{l+d+1}\]$上的部分. 相反地, 区间$\[u\_{l}, u\_{l+1}\]$对应部分地曲线只受$\mathbf{c}\_{l-d-1},\dots,\mathbf{c}\_{l}$这$d+1$个控制点的影响.

我们直到Bezier曲线是通过第一个和最后一个控制点的, 但是B样条曲线却不一定如此. 为了让B样条也能通过第一个和最后一个控制点, 节点向量需要一个特殊的布局. 这个特殊的布局就是之前提到的边界上的节点的重数为$d+1$. 简单地说就是开始的$d+1$个节点必须相等,最后的$d+1$个节点必须相等.

对于不通过第一个和最后一个控制点的B样条曲线, 其定义域为$[u\_{d+1},u\_{d+n+1}]$.

# 节点插入和De Boor算法(Knot Insertion and de Boor’s Algorithm)

如果向节点序列$\mathbf{u}=\dots \leqslant u\_{k} < u\_{k+1} \leqslant \dots$中插入一个节点$u^{* } \in \[u\_{k},u\_{k+1}\)$, 那么将获得一个新的节点序列$\mathbf{u}^{* }=\dots \leqslant u\_{k} \leqslant u^{* } < u\_{k+1} \leqslant \dots$, 且样条空间$S^{d}\_{\mathbf{u}} \subseteq S^{d}\_{\mathbf{u}^{* }}$. 对于节点向量为$\mathbf{u}$的样条曲线$\mathbf{r}(u)=\sum\mathbf{c}\_{k}N^{d}\_{\mathbf{u},k}(u)$, 该曲线可以被重写为$\mathbf{r}(u)=\sum\mathbf{c}^{* }\_{k}N^{d}\_{\mathbf{u}^{* },k}(u)$. 新的控制点$\mathbf{c}^{* }\_{l}$可以由$c\_{l}$给出:

$\mathbf{c}^{* }\_{l}=
\begin{cases}
\mathbf{c}\_{l},\  l \leqslant k-d,\\\\\
(1 - \alpha\_{l})\mathbf{c}\_{l-1} + \alpha\_{l}\mathbf{c}\_{l}, \  l=k-d+1,\dots,k-1,\\\\\
\mathbf{c}\_{l-1},\  l \geq k
\end{cases}$

其中

$\alpha\_{l}=\frac{u^{* }-u\_{l-1}}{u\_{l+d-1}-u\_{l-1}}$

De Boor算法反复的插入单个节点, 直到它的重数$\nu=d$, 并且从曲线上能够得到一个点.设样条曲线$\mathbf{r}(u)=\sum\_{k}\mathbf{c}\_{k}N^{d}\_{\mathbf{u},k}(u)$, 其多项式的最高次幂为$d$, 节点向量$\mathbf{u}=u\_{1},\dots,u\_{2d+n+1}$, 控制点为$\mathbf{c}\_{1},\dots,\mathbf{c}\_{n+d}$. 如果$u \in \[u\_{l}, u\_{l+1}\]$, 则曲线$\mathbf{r}(u)$上的点能够通过De Boor算法得到.

令$\mathbf{c}^{0}\_{i}=\mathbf{c}\_{i}$, 其中$i=l-d,\dots,l$, $k=1,\dots,d$, 循环计算:

$\alpha^{k}\_{i}=\frac{u-u\_{l}}{u\_{l+d}-u\_{l}}$

$\mathbf{c}\_{i}^{k}=(1-\alpha^{k}\_{i})\mathbf{c}^{k-1} + \alpha^{k}\_{i}\mathbf{c}\_{i}^{k-1}$

循环结束后得到$\mathbf{r}(u)=\mathbf{c}\_{l}^{d}$

# 可微性(Differentiation)

最高次数幂为$d$的样条曲线, 其求导后得到的是一条最高次数幂为$d-1$的样条曲线.

B样条$N^{d}\_{\mathbf{u},l}$的导数为

$\frac{d}{du} N^{d}\_{\mathbf{u},1}(u) = -\frac{d}{u\_{2+d}-u\_{2}} N^{d-1}\_{\mathbf{u},2}(u)$

$\frac{d}{du} N^{d}\_{\mathbf{u},l}(u) = \frac{d}{u\_{l+d}-u\_{l}} N^{d-1}\_{\mathbf{u},l}(u) - \frac{d}{u\_{l+d+1}-u\_{l+1}} N^{d-1}\_{\mathbf{u},l+1}(u),\  l=2,\dots,n+d-1$

$\frac{d}{du} N^{d}\_{\mathbf{u},n+d}(u) = \frac{d}{u\_{n+2d}-u\_{n+d}} N^{d-1}\_{\mathbf{u},n+d}(u)$

则对于样条曲线$\mathbf{r}(u)=\sum\_{l}\mathbf{c}\_{l}N\_{\mathbf{u},l}^{d}{u}$

$\mathbf{r'}(u)=\sum\_{l=2}^{n+d}\frac{d(\mathbf{c}\_{l} - \mathbf{c}\_{l-1})}{u\_{l+d}-u\_{l}} N\_{\mathbf{u},l}^{d-1}(u)$

注意, 在区间$\[u\_{d+1}, u\_{d+n+1}\]$上, $N\_{\mathbf{u},1}^{d-1}=N\_{\mathbf{u},n}^{d-1} = 0$.

对于节点向量$\mathbf{u}=u\_{1},\dots,u\_{2d+1+n}$的$\mathbf{u}(u)$求导后得到的曲线的节点向量为$\mathbf{u'}=u\_{2},\dots,u\_{2d+n}$, 且控制点为

$\mathbf{c'}\_{l}=\frac{d(\mathbf{c}\_{l} - \mathbf{c}\_{l-1})}{u\_{l+d}-u\_{l}}$

# NURBS(Non Uniform Rational Basis Spline)
