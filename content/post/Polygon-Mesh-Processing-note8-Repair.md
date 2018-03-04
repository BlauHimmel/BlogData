---
title: "Polygon Mesh Processing 阅读笔记(8) 模型修复(Repair)"
date: 2018-02-28T19:21:17+08:00
draft: false
tags: ["Polygon Mesh Processing", "阅读笔记"]
---
<!--more-->

所谓模型修复，就是取“矫正”模型的一些“畸形”的地方(Artifacts)。

一些比较常见的“畸形”的情景有：

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-0.png)

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-1.png)

一般我们将模型修复算法粗略的分为下面两类：

#### Surface-Oriented Algorithm

这类算法通常直接对输入的数据进行操作并通过修改曲面直接将“畸形”的地方(Artifacts)“矫正”。

如果是“沟(Gap)”，我们可以通过调整边界处的顶点和边的位置将两条边界边界“缝合”起来。

如果是“洞(Hole)”，我们将空白的部分进行三角剖分，从而补上它。

如果是“相交(Intersection)”，我们可以将相交出的三角形和边拆分(Split)。

算法只会额外添加很少的三角形。且不会修改其它正常区域。与此同时还能够较好地保留和顶点、三角形相关联的属性(连接性、纹理等)。

为了保证有效的输出，该算法对输入的模型会有一定的要求，故在执行算法之前和之后还需要我们进行一些手动处理。此外，由于存在精度上的问题，诸如 **相交(Intersection)** 和 **大范围重叠(Large Overlap)** 之类的地方不能被很好地修复，其它的例如在两个非常接近的曲面间产生的 **沟(Gap)** 往往不能够被检测到。

#### Volumetric Algorithm

该算法将输入的模型用测定体积的方式去表示它(简单地说就是该模型占据了空间的哪些区域)。然后根据前面的表示形式，从中提取输出的模型。

所谓测定体积的方式，即将空间划分为一小块一小块，对于每一个小块我们赋予其下面三种状态之一的状体：模型内、模型外和与模型相交。

常见的方法有：根据笛卡尔坐标进行划分、自适应八叉树、KD树、BSP树和3D德洛内三角剖分。

该算法不需要手动进行处理，且输出的模型是完全严丝合缝的(Watertight Model)，能够修复除了 **Handle** 外的其它类型的“畸形”。不过这种类型的“畸形”也可以通过一定方法移除掉。

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-2.png)

从整体上看，算法相当于是对输入的模型进行了一次重采样，因而会造成诸如走样(Alias)、丢失模型特征的问题，并且模型上基于原有网格连接性的相关属性信息也会丢失。因为输出模型的三角形的数目会比输入模型的要多，所以还需要对输出的模型进行简化。并且算法是比较耗费内存的，所以往往很难再高分辨率的模型上运行。

### 输入类型

下面介绍一些常用的模型输入类型以及该类型通常会产生的“畸形”的类型。

#### Registered Range Scan

Registered Range Scan是一系列相互重叠的面片，这些面片可以用来表示输入模型。在将这些面片融合为一个单一的三角形网格的时候就由可能出现问题。由于输入数据中存在较大的重叠，所以曲面上的某一个点$\mathbf{x}$会被若干个面片表示。并且每一个面片都有自己的连接性属性，这些属性和其它的面片并不兼容。

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-3.png)

#### Fused Range Scan

Fused Range Scan是带边界(如Gap、Hole和Island)的流形(Manifold)网格。这些边界是由于扫描仪的实现上出现了遮挡物或者物体表面上的一些特殊属性，如透明或者高光造成的。我们的目标是找到并填补上这些洞(Hole)。更加高级的算法不仅仅是填上这些洞(Hole)，而且还根据该区域周围的几何特征为这个区域合成新的几何特征。

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-4.png)

#### Triangle Soup

Triangle Soup是三角形的集合，并且几乎没有这些三角形的连接性信息。Triangle Soup通常出现在CAD建模中，由用户手动建立(通过一些已经定义好的元素来创建出目标模型的边界)。模型虽然只是由三角形构成，却可能出现各种类型的“畸形”。其中“相交(Intersection)”是最为常见的，对其的检测过程非常耗时。所以该类型通常多用于可视化而不是几何处理。

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-5.png)

#### Triangulated NURBS Patch

Triangulated NURBS Patch是一系列三角形网格的集合，最终的模型由这些网格拼接得到。所以在将不同的网格拼合的时候，拼合处就可能会出现“沟(Gap)”和小的“重叠(Overlap)”。并且在结合处还有可能出现法线方向不一致的情况。通常我们使用**Surface-Oriented Algorithm**来修复它。首先要确保输出的每一块网格的朝向一直，然后在将相邻两块网格咬合(Snap)在一起。

#### Contoured Mesh

Contoured Mesh是通过体积测定数据集(Volumetric Dataset)导出的网格模型。理论上导出的模型是一个流形(Manifold)，并且是密闭的(Watertight)。然而也会出现像下图那样的“畸形”(书中称之为 **Small Spurious Handle** )。

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-6.png)

体积测定数据(Volumetric Data)通常出现在构造实体几何(Constructive Solid Geometry)中，或者在医学图像(CT，核磁共振等)中用作某种中间表示形式。在数据集中的每一个点通常会被赋于一个标量值，负值代表在物体内部，正值代表在物体外部，零值则代表在物体表面上，这些点通常也被称为 **体像素(Voxel)**。

由于分辨率有限，体像素通常不能够被正确的分类(内部、外部还是表面上)，导致 **Partial Volume Effect** ，简单的说就是因为显示粒度相对于数据集中的点来说太大或者太小了导致无法对其进行正常的分类。举个例子，我们知道大脑皮层与球面同胚(能够展开为球面)，但是在我们通过CT扫描的数据导出的模型中一些实际上是分离的地方往往都糊成了一团。

对于 **Handle** 类的“畸形”，我们往往是在原数据集进行处理以修复它，然后在提取出结果模型。然后在对模型进行处理以修复其它的“畸形”。

#### Badly Meshed Manifold

Badly Meshed Manifold是指包含了退化元素(Degenerate Element)的网格，如：

* 面积为$0$的三角形
* 有一个内角为$\pi$(称为 **Cap** )
* 有一条边的长度为$0$(称为 **Needle** )
* 相邻三角形的法向量的夹角接近$\pi$(称为 **Triangle Flip** )

改善这类网格的方法一般是 **网格重划分(Remeshing)**。

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-7.png)

##  Surface-Oriented Algorithm

Surface-Oriented Algorithm直接对输入的网格进行修改，以对其进行显示修复。

###  Surface-Based Hole Filling

这里介绍的补洞算法是其它类似算法的基础。我们的目标是得到一个三角形网格，网格的边界是一个给定的多边形$\mathbf{p}_0$,...,$\mathbf{p}\_{n-1}$，该多边形能够正好放入待修复的“洞(Hole)”中。随后我们还需根据一些与网格属性(如面积，三角形法线的变化程度，曲率分布等)相关的质量函数去优化这块网格。

假设$\psi(i, j, k)$是定义在三角形$(\mathbf{p}_i, \mathbf{p}_j, \mathbf{p}_k)$上质量函数，$\omega (i,j)$是多边形$\mathbf{p}_0$,...,$\mathbf{p}\_{n-1}$的子多边形 $\mathbf{p}_i$,...,$\mathbf{p}\_j$经过三角剖分后得到网格的最优质量，$\omega (i,j)$可以通过下面的迭代式递归的计算出来

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-8.png)

其中整体的最优质量$\omega (0, n-1)$能够通过动态规划算法计算得到。

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-9.png)

函数$\psi(i, j, k)$应该考虑下面两个因素：

* $\Delta_{ijk}$和其周围三角形组成的二面角(等价于法向量的变化程度)
* $\Delta_{ijk}$的面积

{{< figure src="/img/post/Polygon-Mesh-Processing-note8-Repair/img-10.png" title="α是最大的二面角，A是△ijk的面积" >}}

并且第一个因素的决定程度更高，所以

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-11.png)

可以发现，我们在评估一个三角形的质量的时候考虑到了其周围的三角形。这些三角形可能是原来已经有的，也可能是新创建出来。

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-12.png)

在填补完成后，我们还需要对填补上去的网格进行一些调整，使得其顶点分布的密度以及边的平均长度于其周围的网格相匹配。

算法构建三角剖分的时间复杂度为$O(n^3)$，由于“洞(Hole)”一般不大，所以这是可以接受的。然而算法并不能“矫正”“自我相交(Self Intersection)”的问题。

### Gap Closing

对于“沟(Gap)”，我们可以将其看成一个特殊的“洞(Hole)”，使用上面介绍算法来进行修复。

另外可以做从点到边的收缩(Vertex-Edge Contraction)操作。假定我们将顶点$v$收缩指边界$e$上，设$c$是$e$上离$v$最近的点，如果$c$是$e$内部的顶点，则将$c$插入到$e$上，并将相邻的那个三角形一分为二。最后再将$v$和$c$合并即可。可以使用$v$到$c$的距离来作为收缩操作的误差。算法还维护着一个优先队列，优先队列存储的是一对顶点和边，优先级使用的是前面提到的误差。然后按优先级从小到大进行相应的收缩操作。

上述算法本身是完备的，且易于实现。如果输入的模型和设定的阈值比较理想，一般能够得到比较理想的结果。不过往往实际情况并非这么理想，由于算法本身是启发式的，所以输出的模型上可能还是会存在各种“畸形”的地方(Artifacts)。所以算法往往被设定为迭代循环的形式，让用户来引导，使算法向着期望的地方进行迭代。

### Topology Simplification

拓扑简化的目的就是为了移除网格上的 **Handle** ，算法的基本思想如下图所示：首先将 **Handle** 沿着绿线切开，切开后模型上会出现两个“洞(Hole)”，然后使用之前提到的算法将这两个“洞(Hole)”给补上。

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-13.png)

给定一个初始三角形$s$，算法通过对原网格执行Dijkstra算法计算得到$s$周围的Geodesic Region $\mathbf{R}_{\epsilon}(s)$。Dijkstra算法在计算得到最短路径的同时，还能得到每一个三角形到初始三角形 *s* 的最短路径上的上一个三角形(Parent Triangle)，记作$p(t)$，$p(t)$的上一个三角形记作$p^2(t)$...依此类推，路径上的最后一个点为$s$ 。

$\mathbf{R}\_{\epsilon}(s)$ 包含一个或多个环状边界，当有一个环状边界在某一条边上触碰到自己的时候，就将它分裂为两个新环。如果是两个不同的环在$e_{12}$处相互触碰的时候就表示我们检测到了 **Handle** 。假设$t_1, t_2$是与$e\_{12}$相邻的两个三角形，假设$t_1$和$t_2$存在一个共同的祖先

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-14.png)

则该 **Handle** 可以由下面的闭合路径确定

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-15.png)

然后我们沿着该路径将模型切开，并补上新产生的两个“洞(Hole)”。

为了检测原网格中的所有 **Handle** ，我们必须对所有的三角形$s$执行上面的算法，所以算法的效率不高。并且算法不能检测到长的(long)、薄的(thin) **Handle** ，并且不能保证在 **Handle** 被移除后不存在几何自相交的情况。

##  Volumetric Repair Algorithms

###  Volumetric Repair on Regular Grids

这种方法能够修复包含有 **Gap** 、 **Overlap** 以及 **Intersections** 的模型，对于 **Hole** 和 **Handle** ，该方法使用形态学操作(Morphological Operators)来对其进行处理。

算法分为两步，第一步是进行体素化。首先生成 **投影方向集合** $\\{\mathbf{d}_i\\}$(如通过细分八面体或者二十面体)。接下来将模型沿着正交平面网格(Orthogonal Planar Grid)的方向进行投影。对于网格上的每一个点$\mathbf{x}$ ，算法会记录射线$\mathbf{x} + \lambda \mathbf{d}_i$与输入模型的第一个和最后一个交点。位于两者之间体像素(voxel)被分类是处于内部点，其它的体像素(voxel)被分类为外部。每一个体像素会被不同的射线进行分类，最终的分类结果按照少数服从多数的原则选取。

第二个步骤是可选的，通过形态学操作(Morphological Operators)将 **Handle** 和 **Hole** 进行修复。这个过程会用到 **膨胀(Dilation)** 和 **腐蚀(Erosion)** 操作。对于膨胀操作，考虑被分类为内部的顶点组成的集合，将所有与该集合距离小于特定值的外部体像素的分类设置为内部。而腐蚀操作则与膨胀操作正好相反。

该算法是启发性的，通常不是非常的可靠，并且不是特征敏感(Feature sensitive)的，即某些特征可能在修复后丢失。

###  Volumetric Repair on Adaptive Grids

可以使用八叉树代替规则网格以改进上述算法。在改进的算法中会使用下面两个限制变量：容忍度ε和最大修复直径$\rho$(描述需要修复多宽的Gap)。

首先使用八叉树对输入的模型进行划分，$\epsilon$在这里表示节点的最小直径。接下来对八叉树子节点进行分类(处于内部还是外部)。然后对处于模型边界上的节点进行 **膨胀(Dilation)** 操作，膨胀的距离为$ n = \frac{\rho}{\epsilon}$，这样直径小于$\rho$的Gap将会被修复。下一步，从外部向内部进行 **膨胀(Dilation)** 操作，以抵消前一步带来的模型体积的增大。

{{< figure src="/img/post/Polygon-Mesh-Processing-note8-Repair/img-16.png" title="紫色方格代表边界上的节点，绿色表示膨胀边界过程中影响到的节点，橙色表示从外部向内部膨胀过程中影响到的节点" >}}

接下来使用Dual Contouring算法通过采样点进行重建工作即可。采样点通常取与三角形所在平面的平方距离最小值的点(模型简化那一章提到过)，如果找不到这样的平面(节点是通过膨胀操作得到的)，则可以通过平滑算子(Smoothing Operator)来采样得到采样点。

改进算法能够保证输出是一个流形(Manifold)，并且特征能够得到保留，但是重建的分辨率是有限的。

###  Volumetric Repair with BSP Trees

该算法使用BSP树，每一个输入多边形(三角形)所在的平面作为其划分平面，树的子节点对应了一个凸多边形$C_i$。对于每一个凸多边形$C_i$，定义并计算出一个从-1到1的 **硬度系数(Solidity Coefficient)** $s_i$ 。-1表示区域为空(不属于网格内部)，1表示则表示其属于区域内部。

所有无限大的区域都可以被看作是处于物体的外部，因此其 **硬度系数(Solidity Coefficient)** 的值为-1。

设$C_i$为有限区域，$N(i)$表示于其相邻的面的集合。那么对于$j \in N(i)$，交集$P\_{ij} = C_i \cap C_j$表示两区域相交部分多边形所在的平面，$t\_{ij}$表示区域边界平面上不被输入多边形(三角形)包含的部分的面积(称之为透明的)，$o\_{ij}$表示区域边界平面上被输入多边形(三角形)包含的部分的面积(称之为不透明的)，$a\_{ij}$表示平面的总面积。那么$s_i$与和其相邻的$s_j$的关系可以写成

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-17.png)

其中$A_i$表示区域$C_i$边界的面积之和

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-18.png)

通过观察可以发现如果两个区域相交平面大部分为透明的，则有很大的可能它们都是处于内部或者都是处于外部的。如果大部分区域为不透明的，则有很大可能其中一方处于内部而另一方处于外部。

上面的式子可以写成下面的矩阵的形式

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-19.png)

最终通过枚举所有的区域$C_i$，如果其中一方是处于内部的而另一方是处于外部的，则将其边界$P\_{ij}$加入到重建的过程中。

该算法不需要用户指定一些额外的参数，并且可以得到严丝合缝的模型(Watertight Model)。得到的结果中可能包含有 **Complex Edge** 和 **Singular Vertex** (参考开篇的图片)，这些可以使用前面提到的方法进行修复。不过，很难找到即具有鲁棒性又具有高效性的的BSP结构。

{{< figure src="/img/post/Polygon-Mesh-Processing-note8-Repair/img-20.png" title="BSP树(左)，硬度系数(中)，重建(右)" >}}

###  Volumetric Repair on the Dual Grid

该算法首先使用坐标系网格所有面的一个子集$F$来近似输入模型，为了节约内存，可以使用八叉树来辅助这个过程。同时将采样点(和法向量)附在每一个面上，以方便后面进行一些更精确的操作。

对于子集$F$，其边界$\partial F$可以定义为指向奇数个面的坐标系网格边的集合(如下图中红色圆点)。

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-21.png)

对于每一个边界上的回路$B_i$，接下来我们寻找另一个集合$G$，集合$G$与$B_i$相同，且它们的对称差集的边界为空集。然后将$F$进行替换，直到$F$的边界为空集。

![](/img/post/Polygon-Mesh-Processing-note8-Repair/img-22.png)

最后只需要使用Marching Cubes或者Dual Contouring算法进行最后的重建工作即可。

该算法能够保证输出是一个流形(Manifold)，并且算法的内存效率较高能够适应较大的数据量。但是在一些特殊结构，如待修复的 **Hole** 与输入的几何机构存在重合的时候，网格可能会变成几个不相连的部分。
