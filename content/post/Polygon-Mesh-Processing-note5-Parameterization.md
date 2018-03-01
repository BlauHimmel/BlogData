---
title: "Polygon Mesh Processing 阅读笔记(5) 参数化(Parameterization)"
date: 2018-02-28
draft: false
tags: ["Polygon Mesh Processing", "阅读笔记"]
---
<!--more-->

参数化的主要目标是将复杂的3维模型转换到2维空间上。对于一个三角形网格模型，就是把它拍平到一个平面上。

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-0.png)

从数学上解释，对三角形网格参数化的过程就是寻找一种映射，它能够把网格的每一个顶点$i$映射到$(u_i,  v_i)$上。需要注意的一点是，经过映射后，任意两个三角形公共的部分只有可能是：一条边，一个顶点或者不存在。

## 重心映射(Barycentric Mapping)

重心映射是一种在三角化网格中比较常用的参数的方法。使用这种方法的网格必须满足下面的条件：

* 三角形网格与圆盘是同胚的(简单地说就是网格必须有边界并且不能有洞)，并且网格的边界在一个凸多边形上，同时内部的顶点是其周围顶点的凸组合。

假设顶点的索引按照{内部的顶点$(1...n)$，边界上的顶点$(n+1...N)$}排列，那么，对于任取实数$\lambda$，满足：

{{< figure src="/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-1.png" title="其中(i,j)∈E表示点i和点j相邻" >}}

然后将处于边界的点(索引为$n+1$到$N$的点)固定，用下面的线性方程去更新处于内部的顶点：

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-2.png)

写成矩阵的形式就是要分别求解下面两个方程

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-3.png)

而矩阵$A$的元素$a_{ij}$的取法有一般常见的有三种，其中最简单的一种是当下标$i$和$j$不相等的时候取$1$，下标$i$和$j$相等的时候取顶点i周围顶点个数的相反数。这种取法的优势是简单并且计算较快，缺点是没有考虑到网格的集合属性，例如边的长度和三角形的角度，所以会不可避免地产生形变。

另一种方法就是使用下面的cotangent形式

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-4.png)

式中各项的含义如下图所示

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-5.png)

如果$\alpha+\beta > \pi$，那么cotangent形式就有可能出现负数。可以通过将网格面进行分割来解决这个问题，也可以使用下面这种取法来解决

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-6.png)

其中$\gamma$和$\delta$的含义如下图所示

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-7.png)

无论使用上面哪种方法，都需要手动指定并固定网格的边界，但是对于一个模型往往选取边界不是一件容易的事情。边界的选取与固定会影响到参数化的结果。

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-8.png)

## 保角映射(Conformal Mapping)

对于保角度映射，引用之前用到的例子：假设参数空间$(u,v)$上存在一个小圆，使用保角映射将它变换到3维坐标空间之后，其沿着u和v方向的切向量$\mathbf{x}_u$和$\mathbf{x}_v$的模长应该同样地是相等并且$\mathbf{x}_u$和$\mathbf{x}_v$正交。对于此变化的逆变化，即从3维坐标空间变换为2维参数空间也同样成立。

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-9.png)

与之前讨论梯度时的情况不同，参数化的函数是一个从3维曲面到平面的映射。

为了定义这个函数的梯度，对于每一个三角形，选择一个顶点，并以该顶点为原点建立一组正交基

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-10.png)

在这样一组基向量下，函数的梯度的定义是：

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-11.png)

其中对于每一个三角形矩阵$\mathbf{M}$是一个常数，$A$代表了三角形的面积。由于保角映射的逆映射同样也是保角的，并且参数空间下$u$和$v$分量的基向量是垂直的，所以：

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-12.png)

也可以用矩阵的形式来表示

{{< figure src="/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-13.png" title="垂直符号代表向量沿着顺时针的方向以法向量为轴旋转得到的新向量" >}}

然后将之前求得的梯度带入到上面的式子中，得到：

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-14.png)

在连续集上，黎曼几何阐述了一个事实：任何的曲面都能够进行保角参数化。

但是，在分片线性函数表示的三角形网格中，只有 **可展曲面(Developable Surface)** 才能进行保角参数化。对于一般的曲面(非可展曲面)，上式子就不能成立，即梯度向量$u$和$v$不一定是垂直的，但是要尽可能地让它们垂直，所以将上面的式子改写为下面的最小二乘的形式

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-15.png)

上面的方程被称为 *保角能量(Conformal Energy)* ，注意到保角能量的值不会受到参数空间的平移和旋转变换的影响，所以并不存在唯一的最小解，一般在实际应用的时候会 **至少固定两个点** $(u, v)$的位置，然后再去求其最小值。

---

在之前进行曲面平滑(Smoothing)的过程中，为了让曲面变得更平滑，在边界固定的情况下考虑最小化曲面的面积。由于求曲面的面积的方程中包含了平方根和第一基本型，是高度非线性化的，为了简化运算引入了 *狄利特雷能量(Dirichlet's Energy)* 的概念。

而在保角映射中使用到了 *保角能量(Conformal Energy)* 的概念，当保角能量为0的时候说明该参数化的过程是保角的。

而上述的3个量存在下面的关系

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-16.png)

这个式子很容易推导，只需要将等号两边的积分展开就能证明。

曲面平滑的过程中最小化了狄利特雷能量，而在上面的保角映射的过程中使用了最小二乘的思想来最小化保角能量。因为这两个量之间只相差了曲面面积，所以从某种角度上来说，这两种方法是等价的。

## 基于几何方法的保角映射

以上所讨论的都是基于分析的方法，基于分析的方法是易于实现的，因为它需要的只是最小化一个二次的方程。

但是上面的方法均需要固定至少两个顶点，而这将可能会导致变形，如下图所示

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-17.png)

对于一个高斯曲率较高的曲面，上述包角变换的方法往往会造成诸如左图那样的变形，而用户期望的结果一般是像右图那样的。

下面介绍一种基于角度的拍平化算法—— **angle-based flattening(ABF)**。

由于参数空间是一个2维的三角剖分(Triangulation)，并且由某一个顶点周围所有的角唯一地定义，所以可以重新定义解决参数化的问题的思路，即通过角来寻找顶点的坐标$(u, v)$。

ABF方法是通过最小化下面的能量来寻找参数化的结果的：

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-18.png)

其中$\alpha$与$\beta$项表示与顶点$k$相邻的，处于三角形$T$中角度的大小，$\alpha$表示的参数化平面上的角度而$\beta$表示的是原来3D网格上对应位置的角度。

为了保证得到的角度$\alpha$所定义的三角剖分是合法的，需要对其施加一定的限制条件：

* 同一个三角形的内角和必须为180°

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-19.png)

* 对于处于内部的顶点，其周围相邻的角的度数之和为360°

{{< figure src="/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-20.png" title="Vint表示非边界上的点的集合，v*表示与顶点v相邻角度的集合" >}}

* 对于处于内部的顶点，其周围相邻的角对应所处的三角形中的下一个角的正弦值的积等于其周围相邻的角对应所处的三角形中的上一个角的正弦值的积(要统一好三角形中角的顺序)。

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-21.png)

## 基于形变分析的方法

进行纹理映射的时候，如何减小形变是一个非常重要的问题。

首先考虑这么一个问题，两个切向量从参数空间变换到曲面上的时候，量向量的夹角一般会用下面的方法进行计算

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-22.png)

当第一基本型是单位矩阵的倍数或者奇异值相同的时候，向量的角度得到了保留。当所有的顶点都这样的时候，这个变换就可以说是 **保角(angle-preserving or conformal)** 的。

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-9.png)

这个时候，圆形变换之后还是圆形，不过它的半径可能会发生改变。

通过观察计算曲面某区域面积的时候用到的公式

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-24.png)

可以发现当第一基本型行列式的值为$1$或者所有奇异值相乘结果为$1$的时候，圆形的面积在变换后不会发生变化，这个时称变换是 **保面(area-preserving or equiareal)** 的。

当上述条件同时满足，即第一基本型是单位矩阵或者奇异值为1的时候，则称变换是 **等距(length-preserving or isometric)的** 。因为在这种变换中角度和面积都没有发生变形，是一种比较理想的情况，只有一些特殊的曲面，如 **可展曲面(developable surfaces)** 才能满足，因为这种曲面的高斯曲率处处为零(参考：[Theorema Egregium](https://en.wikipedia.org/wiki/Theorema_Egregium)，[高斯绝妙定理](https://baike.baidu.com/item/%E9%AB%98%E6%96%AF%E7%BB%9D%E5%A6%99%E5%AE%9A%E7%90%86/18910263?fr=aladdin)，[相对论与黎曼几何-4-内蕴几何](http://blog.sciencenet.cn/blog-677221-818928.html))。

---

### 如何计算第一基本型？

在进行保角映射的时候，我们定义了参数空间下的梯度，利用它可以得到此映射下的雅可比矩阵

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-25.png)

利用雅可比矩阵，可以得到第一基本型

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-26.png)

并且，对于每一个三角形$T$来说，雅可比矩阵$\mathbf{J}$和第一基本型$\mathbf{Ⅰ}$都是常量。

---

### 两个具体的方法

#### MIPS

下面介绍一种基于形变分析的方法 **MIPS (most isometric parameterization of surfaces)** 。这种方法的主要思想是减小变换过程中小椭圆的两个轴长$\sigma_1$和$\sigma_2$的比值

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-9.png)

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-28.png)

对上式进行最小化相对来说比较复杂，可以将式子中的2范数改成Frobenius范数([矩阵范数](https://en.wikipedia.org/wiki/Matrix_norm)，[Frobenius范式](http://mathworld.wolfram.com/FrobeniusNorm.html))，然后将其化简可以得到

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-29.png)

其中第一基本型的迹可以看作是狄利特雷能量(Dirichlet energy)，而雅可比矩阵行列式的值可以看作是3维空间和参数空间下三角形面积之比。

#### Signal-Specialized Parameterization

这个方法是基于纹理映射的，一般我们将纹理从参数空间映射到曲面上，而纹理变形的程度可以通过观察纹理上的某一个点在某一个方向上被拉伸的程度。

对于某一个特定的三角形，一般使用下面的式子来表示其在所有方向上平均被拉伸的程度

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-30.png)

然后将网格上所有三角形按照面积加权组合一下，就可以得到

![](/img/post/Polygon-Mesh-Processing-note5-Parameterization/img-31.png)
