---
title: "Polygon Mesh Processing 阅读笔记(1) 曲面的表示"
date: 2018-02-28
draft: false
tags: ["Mesh", "阅读笔记"]
---
<!--more-->

大体上看，有两种表示一个曲面的方式：**参数方程** 、 **隐式方程** 。

对于一个3D物体：

1. **参数方程** 是一个从 **2维参数** 到表示3D物体平面上点的空间坐标的 **3维参数** 的一个 **映射** 。

2. **隐式方程** 是一个等号左边为表示3D物体平面上点的空间坐标的 **3维参数** 构成的标量表达式，等号右边为0的方程。

以平面上的单位圆为例，它的参数方程和隐式方程如下：

{{< figure src="http://upload-images.jianshu.io/upload_images/6808438-41f76f59db839150.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" title="参数方程" >}}

{{< figure src="http://upload-images.jianshu.io/upload_images/6808438-6f61bafc777a5227.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" title="隐式方程" >}}

在实际中，一般根据实际需求采用不同的表示方法，这些需求一般分为如下3点：

1. **Evaluation**：在对曲面进行采样的时候需要对其附加上一下除了空间信息之外的信息。例如在进行渲染的时候，除了几何体的空间坐标外，还需要其法向量信息。

2. **Query**：一个典型的空间查询是判断空间中的某个点是否在几何体的内部，另外还比如空间中某个点到某个曲面的距离。

3. **Modification**：一个曲面可以在 **几何** 上被修改（将一个平面卷起来），还可以在 **拓扑** 关系上被修改（把几张纸合并成一张更大的纸或者挖掉一张纸的一部分）。

## 曲面的定义和属性

书中对于曲面的定义如下：

>an orientable continuous 2D manifold embedded in $ R^3 $

个人的理解是：处于3维空间中的，方向连续的，不存在通过无限细小特征连接的的部分。

以下面两种情况为例，无限细小特征指的两个正方体表面相连接的部分。

{{< figure src="http://upload-images.jianshu.io/upload_images/6808438-6568eccb4a55b45c.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" title="通过点相连接" >}}

{{< figure src="http://upload-images.jianshu.io/upload_images/6808438-b712402774ee43c4.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" title="通过面相连接" >}}

这样两个正方体组成的组合体的外表面就不能称为一个面。

这样的组合体可以称之为：*degenerate 3D solid*

而把上面的两个组合体连接的部分变成下面这个样子之后，就不存在所谓的“通过无限细小特征相连接”的情况了。

![](http://upload-images.jianshu.io/upload_images/6808438-f67bcada8986fc73.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/6808438-84be24ffb1f6c23c.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样组合体可以称之为：*non-degenerate 3D solid*

## 点边面的关系

在一个闭合连接网格内点(Vertex)，边(Edge)和面(Face)的数量存在下面的关系：

> Vertex - Edge + Face = 2(1 - *g* )

上面的式子中*g*表示的是曲面的亏格(*genus*)，它的标准解释为：

>若曲面中最多可画出n条闭合曲线同时不将曲面分开，则称该曲面亏格为n

而对于一般的**闭合曲面**来说亏格n就是几何体上洞眼的数目

{{< figure src="http://upload-images.jianshu.io/upload_images/6808438-8324322c1a448d54.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" title="n = 0(左) n = 1(中) n = 2(右" >}}

对于一个三角形网格，基于上面的式子还能得到下面的关系：

>1. 三角形的数量大约是顶点数量的2倍 -> Face ≈ 2 * Vertex
>2. 边的数量大约是顶点数量的3倍 -> Edge ≈ 3 * Vertex
>3. 顶点邻边的数量平均是6
