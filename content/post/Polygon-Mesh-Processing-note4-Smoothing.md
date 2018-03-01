---
title: "Polygon Mesh Processing 阅读笔记(4) 曲面的平滑(Smoothing)"
date: 2018-02-28
draft: false
tags: ["Mesh", "阅读笔记"]
---
<!--more-->

一般有两种曲面平滑的方式:

* **Denoising**：一般是去掉凸出曲面的部分(高频部分)，而保留和曲面相当的部分(低频部分)。即需要一个在离散三角形网格曲面上的低通滤波器(low-pass filters)，并且需要建立 *频率* 的相关概念。

* **Fairing**：在平均(个人认为翻译成抹平更形象一些)的过程中，所做的不仅仅只是去除高频的部分。抹平的过程相当于是对曲面做了一个变换，使其从各个角度(曲率、高阶导数)上看尽可能的光滑。

## 傅里叶变换

傅里叶变换表示一种映射$f$ ，它将函数从 *空间域(spatial domain)* $f(x)$变换到 *频域(frequency domain)* $F(\omega)$

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-0.jpeg)

上式中的指数部分通过欧拉公式可以展开成如下的复数形式：

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-1.jpeg)

这是一个包含有正弦和余弦函数的，以$\omega$为自变量的(可以看作是频率)函数，可以将这个函数看做向量空间上的一组正交基，即 **频域(frequency domain)**。

可以将$f(x)$(可积的复数函数)看作向量空间中的一个元素，然后对它做下面的内积运算：

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-2.jpeg)

那么，傅里叶变换在这里表示了一种基的变换。通过将向量$f$投影到不同频率的基向量上，然后对其进行累加操作，这样就完成了从 **空间域(spatial domain)** 到 **频域(frequency domain)** 的转换。

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-3.jpeg)

如果我们要除去频率比较高的部分，保留

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-4.jpeg)

只需要将上面的累加式改写为如下的形式

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-5.jpeg)

---

对于离散的三角形网格，需要将连续的函数$f(x)$用如下逐顶点的矩阵形式来表示

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-6.jpeg)

同样的如果要将 *拉普拉斯一贝尔特拉米算子(Laplace-Beltrami operator)* 应用到函数上同样也需要将变成逐顶点的矩阵形式，这样算子就变成了相应的 *拉普拉斯一贝尔特拉米矩阵* $\mathbf{L}$了

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-7.jpeg)

根据之前的知识，可以知道，对于上面式子的每一行是按下面的方法进行运算的：

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-8.jpeg)

权重$w_{ij}$取值的时候要保证矩阵$\mathbf{L}$是对称的。前面提到过两种取值的方法：

1.  Uniform形式

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-9.jpeg)

2. Cotangent形式

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-10.jpeg)

---

可以发现，对函数eω是拉普拉斯算子的特征函数，因为：

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-11.jpeg)

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-12.jpeg)

这样1维傅里叶变换中的基就是 *拉普拉斯一贝尔特拉米算子* 的特征矩阵，很自然的可以想到，对于2维流形曲面其同样成立。

在处理离散形式的时候，$\mathbf{e}_\omega$就变成了矩阵$\mathbf{L}$的特征向量$\mathbf{e}_1$... $\mathbf{e}_n$，对于$\mathbf{e}_i$，用如下逐顶点的矩阵来表示

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-13.jpeg)

$\mathbf{e}_i$的特征值代表了点$v_i$所处频域的频率， $\mathbf{e}_i(v_k)$表示顶点的振幅。

对矩阵$\mathbf{L}$的所有特征向量进行累加可以得到和前面相似的式子：

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-14.jpeg)

如果要滤掉高频部分，那么只需要对前m个特征向量进行累加即可：

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-15.jpeg)

通过下图可以看到，随着m的逐渐减小，即滤掉的频率越来低，模型的凸出部分(细节信息)在逐渐的消失。

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-16.jpeg)

为了得到特征向量需要对拉普拉斯矩阵进行特征分解，而当模型的顶点比较多的时候，代价是时分昂贵的。

而下面的Diffusion Flow方法则相对来说更容易实现，效率也更高一些。

## Diffusion Flow

诸如热扩散和布朗运动之类的物理过程可以使用下面的扩散方程来表示

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-17.png)

从形式上来看，这是一个二阶线性偏微分方程，通常$f(\mathbf{x},t)$表示某点$\mathbf{x}$在时刻的温度，这个方程描述了无体内热运动的规律。

要将其运用到曲面网格上，首先是将连续形式的拉普拉斯一贝尔特拉米算子替换为离散形式，然后将函数$f$改写为逐顶点形式

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-18.png)

为了简介，可以使用矩阵的形式来表示

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-19.png)

等式左边的偏导数可以改写成微商的形式

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-20.png)

化简得到

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-21.png)

为了保证在$h$比较大的情况下的准确性，通常会将上式改写成如下形式：

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-22.png)

然后将自变量相同的移到一边，写成如下矩阵方程：

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-23.png)

最后要做的就是用上面的方程去更新网格的每一个定点

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-24.png)

由于定点的拉普拉斯一贝尔特拉米算子等于其平均曲率法向量

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-25.jpeg)

所以上面的方式实际上是让每一个定点沿着其法向量的方向移动，移动距离由这一点的平均曲率$H$决定。

一个很重要的一点是，只有在算子的权重系数$w_{ij}$的取法是cotangent方法，定点才会向上面说的那样更新。当系数的取法是uniform时，每个顶点会向着重心的方向移动。

{{< figure src="/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-26.png" title="(左)原模型，(中)Uniform，(右)Cotangent" >}}

Diffusion Flow方法比之前使用傅里叶变换来说计算量更低，但是其主要的思想仍然是移除高频噪音(模型上不平滑的地方)而保留低频部分。

## Fairing

Fairing这种方法的思想则与前两种不同，其目标是通过计算使得网格尽可能的光滑。判定光滑程度的方法各不相同，但是总的原则就是要尽可能的光滑，避免不必要的细节或者毛刺。

总的思路是：
1. 定义一个量来衡量曲面的不平滑程度
2. 更新曲面以最小化曲面的不平滑程度

一个比较常用的描述不平滑程度的函数是使用曲面的面积

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-27.png)

在限定了曲面边界的情况下，不平滑的曲面相对的面积应该更大。当其取最小值的时候，外形应该像没有鼓起来的被夹紧的肥皂泡一样。

不过这个函数是高度非线性的，包含了第一基本型行列式的平方根，另外其计算的效率也不会太高。其改进形式如下(参考：[Dirichlet energy](https://en.wikipedia.org/wiki/Dirichlet_energy))：

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-28.png)

为了求最小值，简化上面的模型至一维的情况

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-29.png)

在一维的情况下，限制条件就从边界变成了区间。假设当函数$E$的自变量函数取$f$时$E$取到最小值，并且限制条件的区间为$[a, b]$，那么对于任意函数$u$ ，并且$u(a) = u(b) = 0$，都有$E(f) < E(f+\lambda u)$，那么当$\lambda = 0$时，$E$取最小值，其对于$\lambda$的偏导数的值也为$0$

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-30.png)

用分部积分公式展开上面的积分

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-31.png)

因为$u(a) = u(b) = 0$所以最终

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-32.png)

由于上式对于任意满足$u(a) = u(b) = 0$的函数$u$都成立，所以

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-33.png)

上式推广到二维情况同样试用，所以对于改进后的函数同样可以用这个方法

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-34.png)

为了把上面的方法应用到三角形网格的曲面上，只需要和前面一样将算子和函数改为离散形式即可

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-35.png)

---

除了用面积来衡量不光滑度之外，还可以使用曲面的主曲率：

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-36.png)

或者主曲率相对切向量的变化率：

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-37.png)

当$k = 1$时，以面积为度量的不平滑度取最小值；$k = 2$时以主曲率为度量的不平滑度取最小值；$k = 3$时以主曲率相对切向量的变化率为度量的不平滑度取最小值

![](/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-38.png)

下图中紫色的区域就是通过上面的方法平滑得到

{{< figure src="/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-39.png" title="(左)k = 1，(中)k = 2，(右)k = 3" >}}

算子中权重取法的不同也会产生不同的结果，使用Uniform的取法会导致一些区域的顶点密度过高，使用Cotangent则能够达到期望的平滑效果

{{< figure src="/img/post/Polygon-Mesh-Processing-note4-Smoothing/img-40.png" title="(左)原模型，(中)Uniform，(右)Cotangent" >}}
