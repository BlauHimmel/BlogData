---
title: "Polygon Mesh Processing 阅读笔记(9) 变形(Deformation)"
date: 2018-02-28T19:31:17+08:00
draft: false
tags: ["Polygon Mesh Processing", "阅读笔记"]
---
<!--more-->

一个三角形网格的 **变形(Deformation)** 算法应该满足下面两个基本条件

1. 能够隐藏于交互界面之后
2. 效率足够高以满足交互需求

将曲面S变形为曲面S'的过程可以描述为：给定一个 **位移函数(Displacement Function)** ，该函数输入曲面上的点$\mathbf{p} \in S$，给出一个 **位移向量(Displacement Vector)** —— $\mathbf{d(p)}$ ，并通过以下方式将曲面S映射为变形后的曲面$S'$

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-0.png)

对于离散的三角形网格，位移函数$\mathbf{d}$是分段线性(Piecewise Linear)的，即对于 $\mathbf{p}_i \in S$

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-1.png)

为了人为的控制变形的过程，我们常常会在网格上指定一些控制点$\mathbf{p}_i \in S \subset H$，然后固定网格的一部分$F \subset S$，对于这些点，其位移函数可以描述为

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-2.png)

下图中我们对一个正方形的曲面$S$进行变形，固定曲面$S$浅蓝色的部分$F$，然后选取黄色部分$H$的顶点作为控制顶，将其向上拉动。可以看到经过变形后，没有被固定的部分($R$，即深蓝色部分)的顶点的位置发生了相应的变换。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-3.png)

一个主要的问题就是如何选取合适的位移函数$\mathbf{d}_i$，使得变形的结果符合需求。这里将会讨论两大类变形的方法

1. **基于曲面的变形(Surface-Based Deformations)**：位移函数$\mathbf{d}$是从曲面$S$到三位空间的映射，即计算时在三角形网格上进行的。这类方法能够有很高的控制度，能够对每一个顶点都加以控制，但是其鲁棒性不好，运行效率往往取决网格的的复杂度和质量。

2. **空间变形(Space Deformations)**：位移函数$\mathbf{d}$是从三维空间到三位空间的映射，即对曲面S的变形是隐式的。因为计算不依赖于三角形网格曲面S，所以其不受网格复杂度和质量的影响。

这里讨论的大部分方法都是线性的方法，通常只需要解线性方程，即最小化二次能量能量(Quadratic Deformation Energy)。使用线性系统的优点在于求解的效率高，缺点是有些时候得到的结果是不符合直观的。非线性的方法通过最小化更为精确的变形能量，能够达到更好的效果，但是求解效率确不高。

## Transformation Propagation

一个常用且简单的方法是将对控制点的变换传播到整个变形区域上。在指定好控制点$H$和变形区域$R$后，控制点由用户控制发生变换$\mathbf{T}$，然后将变换$\mathbf{T}$插值传播至变形区域$R$上，使得从固定区域F至变换后控制点所在的区域$H'$的变化是平滑的。

两者间的插值混合可以由一个标量场$s$进行控制

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-4.png)

$s = 1$代表顶点处于控制区域$H$(区域内的顶点被完全变换)，$s = 0$代表顶点处于固定区域F(区域内的顶点不发生变换)，而位于变换区域$R$内的顶点的$s$值则由顶点到区域$F$和区域$H$的距离决定

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-5.png)

距离既可以是测地线距离也可是欧氏距离，前者计算更复杂但是结果的效果更好。

另外标量场$s$也能够是曲面$S$上的调和场(无源无旋)，即其满足拉普拉斯方程

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-6.png)

对于区域$F$和$H$我们加以狄利特雷限制(Dirichlet Constraint)，然后解下列线性拉普拉斯方程即可得到标量场$s$ 。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-7.png)

虽然此法性能逊于前者，但是能够保证结果足够光滑，而前者基于距离的方法只能保证$C^1$连续。

标量场$s$还能够进行进一步的调整以提供更多的控制和灵活度。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-8.png)

得到标量场后，对每一个顶点按以下方法进行插值运算，即可得到变形后顶点的位置。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-9.png)

不过此法存在一个问题，得到结果并不是几何上最直观的结果，还需要对控制区域$H$内的顶点的位移函数$\mathbf{d}$进行平滑处理，或者使用最小化某些基于物理量的变形能量的方法。

{{<figure src = "/img/post/Polygon-Mesh-Processing-note9-Deformation/img-10.png" title = "原模型(左)；使用插值法计算结果(中)；理想的结果(右)">}}

##  Shell-Based Deformation

为了得到更直观准确的结果，位移函数$\mathbf{d}$可以通过最小化基于物理的变形能量的方法得到。我们间曲面S看作是能够 **拉伸(Stretching)** 或者 **弯折(Bending)** 的物理材质(皮肤、布料等)，然后使用能量函数来描述拉伸和弯折的程度。

设参数曲面S和S'，曲面由方程$\mathbf{p}:\Omega \rightarrow R^3$、$\mathbf{p'}:\Omega \rightarrow R^3$给出，且位移函数被定义为 $\mathbf{d}:\Omega \rightarrow R^3$。**第一基本型** 和 **第二基本型** 能够被用来衡量曲面的内在几何量(如长度、面积和曲率等)。当曲面$S$被变形为$S'$时，其基本型由$\mathbf{Ⅰ}$、$\mathbf{Ⅱ}$变为了$\mathbf{Ⅰ'}$、$\mathbf{Ⅱ'}$，它们的差可以用来描述拉伸和弯折(原文中称这种能量为Elastic Thin Shell Energy)

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-11.png)

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-12.png)

刚度参数(Stiffness Parameters)$k_s$和$k_b$被用来控制曲面对拉伸和弯折变换的抵抗程度。在实际应用时只需要在变形的区域最小化上述能量即可。

但是上式由于是非线性的，计算量较大，无法应用的到交互式的程序中。所以通常将基本型简化为位移函数$\mathbf{d}$的偏导数(位置之差)，得到下树的Thin Shell Energy

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-13.png)

其中

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-14.png)

这个函数和前面介绍平面平滑算法里Fairing方法中衡量曲面面积和曲率的能量的函数相类似。区别在于这里我们使用平移量$\mathbf{d}$而不是位置$\mathbf{p}$且最小化的是面积和曲率的 **变换程度** ，即我们最小化曲面的拉伸和弯折。

下图中，我们固定灰色区域，抬升黄色区域，并且最小化Thin Shell Energy。该能量包含拉伸和弯折两个部分，左图展示了纯拉伸的情况$(k_s = 1, k_b = 0)$，中图展示了纯弯折的情况$(k_s = 0, k_b = 1)$，右图展示了两者混合的情况$(k_s = 1, k_b = 10)$。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-15.png)

应用曲面平滑算法里Fairing方法中提到求最小化的方法，得到能量函数对应的欧拉拉格朗日方程

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-16.png)

为了最小化能量，需要解上述的偏微分方程(PDE)，根据第三章中介绍的方法，将上式子写成离散形式，其中

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-17.png)

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-18.png)

则上述偏微分方程(PDE)可以被离散为下面逐顶点形式

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-19.png)

其中变换区域R内自由顶点的位移函数$\mathbf{d}_1$,...,$\mathbf{d}_n$是未知的(方程左侧 $\mathbf{x}$项)，区域$H$和区域$F$中位移函数是已知的(方程右侧$\mathbf{b}$项)。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-20.png)

$\mathbf{L}$为拉普拉斯矩阵，$\mathbf{x}$和$\mathbf{b}$都是n行3列的矩阵。

最小化得到结果是$C^1$连续的，在三角形网格中$C^1$连续只由区域$F$和$H$中First-Two-Rings顶点定义，在最小化的时候不用考虑$F$和$H$区域上其它的顶点。

在交互的过程中会控制区域$H$内的顶点进行操作，使得矩阵方程的右侧的$\mathbf{b}$项不断发生变换，这个情况有更加高效的算法。通过变换限制为仿射变换，也能够将某些计算提前进行预计算以提高效率。

与Transformation Propagation方法相比，此法由于需要每一帧解一个线性方程，计算量相对较大，但是仍然是可交互的。由于此法基于物理法则，故其效果相对较好。

## Multi-Scale Deformation

前文的Shell-Based Deformation方法并不能够正确地处理小尺度的细节。由于对局部细节的旋转变换并不是线性的，所以不能够完全的使用线性的方法对其进行建模。一个更好的方法是使用后面即将介绍的多尺度变形的方法。

下图中，我们抬升正方形曲面的右侧，左二图展示了使用前文中的线性方法得到的结果，发现其细节并没有被正确的还原。使用Multi-Scale Deformation方法得到结果(左三图)虽然仍然有变形，但是已经和理想的情况(左四图)非常接近了。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-21.png)

Multi-Scale Deformation的主要思想是使用在曲面平滑算法中提到的分解的方法将曲面分解为高频和低频两个部分。低频部分即是曲面大致的外形，而高频部分则代表小尺度的细节。我们的目标是对低频部分进行变形并保持高频部分的细节。

这个过程在2维情况下如下图所示，虚线部分表示了曲线的低频部分，我们将这条虚线进行变形并添加上高频细节，最终得到了理想的形变结果。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-22.png)

在三维的情况下，首先通过移除高频部分计算出曲面$S$的低频形式$B$(原模型的光滑简化形式)，在$B$上模型的细节$D$被移除。将$B$形变得到$B'$，通过$B'$和$D$我们能够重建出最终的变形后的曲面$S'$。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-23.png)

上图中我们只对原模型进行了一次分解，同样地也可以对B再一次进行分解，以达到多吃变形的目的。

可以看到Multi-Scale Deformation主要包含下面三个操作

1. 分解(Decomposition)
2. 变形(Deformation)
3. 重建(Reconstruction)

其中分解可以使用到第四章提到的网格光滑算法，变形可以使用前文提到的算法。没有提到的就是如何提取细节D并进行相应的重建。

### 位移向量(Displacement Vectors)

最直接的表示方法就是使用一个向量函数$\mathbf{h}: B \rightarrow R^3$，函数$\mathbf{h(p)}$表示光滑曲面$B$上每一个顶点都对应着一个三维向量。由于$S$和$B$拥有相同的连接性，所以位移向量

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-24.png)

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-25.png)

其中$\mathbf{b} \in B,\mathbf{p}_i \in S$。

使用全局坐标系取表示位移向量得到结果如下图左图所示，正确的方法是使用局部的基向量去表示位移向量(下图右图)。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-26.png)

因此在存储$\mathbf{h}_i$时，需要使用曲面$B$上每个顶点的局部标价下的坐标而不是全局坐标。我们一般取法向量$\mathbf{n}_i$和另外两个向量$\mathbf{t}\_{i,1}$和$\mathbf{t}\_{i,2}$作为一组正交基

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-27.png)

基向量在从$B$变形到$B'$的过程中会发生相应的旋转，最终我们根据$B'$的基向量以及位移向量在$B$中局部坐标基下的坐标可以得到$S'$上每一个顶点的坐标

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-28.png)

法向量$\mathbf{n}_i$在每一个顶点上都是有定义的，剩下的只需要按照统一的标准取另外两个轴$\mathbf{t}\_{i,1}$和$\mathbf{t}\_{i,2}$即可。

### 法向量位移(Normal Displacements)

当位移向量过长的时候会导致结果不稳定，特别是在进行弯折(Bending)变形的时候，因此位移向量应该越短越好。因此，我们想到不再去寻找$B$上$\mathbf{p}_i$的对应顶点$\mathbf{b}_i$，而是去寻找B上距离$\mathbf{p}$最近的顶点。

这种思想就是所谓的法向量位移，即

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-29.png)

因为在上一节中$\mathbf{h}_i$通常是不与法向量平行的，因此法向量位移方法需要对$S$和$B$上的顶点进行重新采样，从$\mathbf{b}_i \in B$上发射一条与法向量平行的射线以找到其在S上对应的顶点$\mathbf{p}_i$，而重采样则会导致Alias Artifacts(走样/假频)现象的出现。

为了改进上面的方法，我们换一个方向。对于点$\mathbf{p}_i \in S$，我们寻找一个点$\mathbf{b}_i \in B$，且$\mathbf{p}_i - \mathbf{b}_i$与$\mathbf{b}_i$的法向量平行，而$\mathbf{b}_i$是曲面$B$上的任意一点，该点处于$B$上某一个三角形$(\mathbf{a}, \mathbf{b}, \mathbf{c}) \in B$之中，因此$\mathbf{b}_i$可以表示为下列重心坐标的形式

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-30.png)

其法向量同样可以由重心坐标插值得到

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-31.png)

而寻找点$\mathbf{b}$的过程，可以使用牛顿迭代法求解下面方程的根

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-32.png)

整个过程大致为，首先寻找离$\mathbf{p}_i$最近的三角形，如果在进行牛顿迭代的过程中重心坐标出现了负值，则分别对其相邻的三角形进行处理。

一旦得到了三角形$(\mathbf{a}, \mathbf{b}, \mathbf{c})$和重心坐标$(\alpha, \beta, \gamma)$，则可以通过变形后的曲面$B'$计算出$S'$上每一个顶点$\mathbf{p}_i$的坐标

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-33.png)

这样避免了对曲面进行重采样，从而使得某些尖锐细小的特征(Sharp Features)得到保留。因为点 $\mathbf{b}_i$是曲面$B$上的任意一点，因此对于曲面$S$和$B$来说，其连接性并不一定要求是已知的。我们可以利用这一点来对曲面$B$进行重采样以获得更高的数值鲁棒性。

位移向量和法向量位移的长度的不同通常取决于曲面$B$和曲面$S$相差的程度，对于例子

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-34.png)

位移向量的长度平均比法向量位移的长度要长9倍。除了长度更短法向量位移也不需要进行启发式的计算(计算坐标基$\mathbf{t}$的过程)。

法向量位移的方法效率极高，但是其主要的问题在于相邻的顶点的位移向量之间并没任何联系。当弯折(Bending)程度较大的时候，会导致细节部分出现非预期的结果。在极端情况曲面可能还能出现自相交的情况(当$B'$的曲率比位移长度$h_i$要大的时候)。

下面的两种算法正是为了改进上述情况而提出的

* **Displacement Volumes**：曲面$S$和曲面$B$对应的两个三角形$(\mathbf{p}_i, \mathbf{p}_j, \mathbf{p}_l)$和$(\mathbf{b}_i, \mathbf{b}_j, \mathbf{b}_l)$组成了一个棱柱体，该棱柱体的体积被用来作为细节$D$，在变形的过程中保持不变。对于变形的曲面$B'$，重建的过程是找到一个曲面$S'$使得其每一个棱柱体的体积都和原来的相等。这样会使得结果根据直观合理且避免了自相交的情况，缺点是该方法在重建的过程中计算量较大。

* **Deformation Transfer**：原书中没有详细介绍该算法，可以参考这篇论文[Deformation Transfer for Detail-Preserving Surface Editing](http://www.citmed.uni-bielefeld.de/publications/vmv06.pdf)。该算法在最终效果和运行效率上取了一个折中，只需要解一个疏松的泊松方程即可。

## Differential Coordinates

尽管前面的Multi-Scale Deformation方法效率高，且能够保存模型中的细节，但是如何生成一个合理的层级结构确是一个相当复杂的过程。为了避免Multi-Scale Deformation中分解的过程，另外一类方法采用了修改曲面的微分属性而不是空间坐标的方法来重建变形后的曲面。

### Gradient-Based Deformation

对于原始网格上每一个顶点上某个标量值，都可以找到对应的分段线性函数

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-35.png)

其梯度是一个常向量(每一个三角形T对应一个常向量)

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-36.png)

同样地，考虑以下三维的情况

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-37.png)

则对应每个三角形$T$，它的梯度是一个3*3的雅可比矩阵

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-38.png)

参考第三章中介绍的方法，可以计算出矩阵中的各个位置上梯度函数的值。

接下来对每一个三角形的梯度$\mathbf{J}_T$进行变形，即乘以一个3*3的变换矩阵(旋转、缩放/错切)$\mathbf{M}_T$

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-39.png)

$\mathbf{M}_T$是根据对控制点的变换得到，具体的方法会在后文中介绍。

剩下的步骤就是在尽量保持每一个三角形梯度不变得情况下，寻找每个顶点的新的位置。

如下图所示，黄色区域为控制点所在区域，原模型为圆柱体表面(左图)，我们对变换区域(蓝色区域)上每一个三角形施以对控制点相同的变换，这将使得模型“裂开”(中图)，然后改变每一个三角形的位置且尽量保持三角形的朝向不变，最终得到了变形后的模型(右图)。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-40.png)

这个过程即是最小化下列的能量函数

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-41.png)

$f$是待寻找的方程，$\mathbf{g}$是目标梯度场。为了最小化上式子，应用变分法解下列欧拉拉格朗日方程

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-42.png)

用目标的的$x$, $y$, $z$坐标代替$f$，并用离散拉普拉斯算子和离散散度算子对原方程离散化得到线性方程

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-43.png)

为了使得方程有解(系统非奇异)，需要将固定一些点$\mathbf{p'}_i$，如示意图中黄色的控制区域H内的顶点(这也就是上面的示意图的中图里黄色区域的控制点的位置被直接修改的原因)、示意图中灰色的固定区域F内的顶点。

解上述方程只需要解一个输送的泊松方程，比前面的Shell-Based Deformation效率略高。另外，泊松方程在边界处只是$C^0$连续的，但是Shell-Based Deformation是$C^1$连续的。

### Laplacian-Based Deformation

Laplacian-Based Deformation与Gradient-Based Deformation方法类似，不过其目标不再是逐三角形的梯度，而是逐顶点拉普拉斯坐标。

我们首先计算每个顶点的拉普拉斯坐标，然后乘以变换矩阵$\mathbf{M}$，最后寻找新的顶点坐标去你和目标拉普拉斯坐标。

这个过程中最小化能量函数为

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-44.png)

对应的欧拉拉格朗日方程为

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-45.png)

上式离散化后得到下列方程

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-46.png)

在解方程的时候同样的需要固定一些顶点。需要注意的是，拉普拉斯算子的离散形式有Uniform和Cotangent两种，对于不规则网格使用后者得到的结果效果会更好一些。

Laplacian-Based Deformation方法和Shell-Based Deformation方法之间是存在联系的。忽略掉Laplacian-Based Deformation中对拉普拉斯坐标的变换，使用原始的拉普拉斯坐标来计算新的顶点的位置，在Shell-Based Deformation中固定相同的顶点且原始顶点相同，两者都能得到相同的欧拉拉格朗日方程

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-47.png)

则最终得到相同的结果。

### 如何得到变换矩阵M

那么前面两种方法中都使用到了变换矩阵$\mathbf{M}$，这一节将会讨论如何根据对控制点的变换得到逐顶点和逐面(三角形)的变换矩阵

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-48.png)

#### Propagation Of Deformation Gradients

第一种方法和之前提到的Transformation Propagation中插值的方法类似，这里我们对变换的梯度进行插值。通常我们是按照下面的方式对控制点进行仿射变换

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-49.png)

$\mathbf{T(x)}$的梯度是一个3*3的矩阵$\mathbf{M}$，该矩阵代表了对控制点的旋转、缩放/错切变换。

需要注意的是，旋转变换的插值与缩放/错切变换不同，需要对变换矩阵进行 **极分解(Polar Decomposition)**。

首先对$\mathbf{M}$进行奇异值分解，得到

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-50.png)

然后就能得到$\mathbf{M}$矩阵中旋转的部分和缩放/错切的部分

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-51.png)

因为$\mathbf{U}$和$\mathbf{V}$是正交矩阵，所以有

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-52.png)

然后我们对旋转部分是[slerp](https://en.wikipedia.org/wiki/Slerp)插值，对缩放/错切的部分使用线性插值，得到逐顶点的变换矩阵$\mathbf{M}_i$

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-53.png)

对于逐面(三角形)，其插值的因子$s$是三角形T的三个顶点对应的值$s_i, s_j, s_k$的平均值。需要注意的是，平移变换$\mathbf{t}$并不会改变梯度和拉普拉斯坐标的值，所以当变换中包含有距离较大的平移变换的时候会产生不符合预期的结果。

#### Implicit Optimization

通过最小化下面能量函数，Implicit Optimization同时对新的顶点坐标$\mathbf{p'}_i$以及旋转矩阵$\mathbf{M}_i$进行优化

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-54.png)

其中$A_i$是顶点的局部面积，$\mathbf{M}_i$和新顶点的位置有关。注意这个过程中同样需要固定区域$H$和区域$F$中的顶点。

为了避免非线性最优化(Nonlinear Optimization)(这是刚体变换$\mathbf{M}_i$中必须满足的)，局部变换被限制为相似线性变换(Linearized Similarity Transformation)，$\mathbf{M}_i$被写成下面的斜对称矩阵

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-55.png)

参数$s_i$和$\mathbf{h}_i$(位移向量)是由下列限制条件决定

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-56.png)

通过$\mathbf{p'}_i$的线性组合可以得到$s_i$和$\mathbf{h}_i$。

书中对该部分介绍省略了很多，详细的内容可以参考这篇论文：[Laplacian Surface Editing](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.360.5276&rep=rep1&type=pdf)

需要注意的一点是，根据拉普拉斯坐标的性质，其对 **旋转敏感** ，所以网格的局部信息会发生旋转扭曲，且旋转尺度较大是，扭曲会非常严重。

##  Freeform Deformation

前面讨论的所有方法都是基于曲面的(Surface- Based)，它们通过最小化某个能量函数在原曲面$S$上进行光滑变形。其计算过程归结起来是解一个线性系统对应的欧拉拉格朗日方程。

上述方法的一个明显的缺点在于其计算量和数值鲁棒性和网格分割的复杂度和质量相关。对于退化的三角形(Degenerate Triangle)其Cotangent形式的拉普拉斯算子是不符合定义的，这会导致线性系统奇异。同样地，Gap和非流形的出现使得顶点地局部信息不再一致，也会导致一些问题。诸如模型修复或者网格重划分算法能够在一定程度上接近这些问题。纵使网格的质量足够高，但是其复杂过大也会导致线性系统规模过大无法得到其解。

接近这些问题的办法是使用 **空间变形(Space Deformations)** ，它通过对目标模型的周围空间进行变形从而隐式的对目标模型进行变形。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-57.png)

与 **基于曲面的变形(Surface-Based Deformations)** 不同，**空间变形(Space Deformations)** 的变形是一个从三维空间到另一个三维空间的过程。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-58.png)

且$\mathbf{d}$不依赖于特定的曲面，能够作用用各种显式表示的曲面(三角形网格的所有顶点、点采样模型的所有点)。

### Lattice-Based Freeform Deformation

Freeform Deformation(FFD)中使用3元张量样条函数(Trivariate Tensor-Product Spline Function)来表示空间变形

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-59.png)

其中$N_i$是B样条函数，$\mathbf{\delta_c}$是控制点$\mathbf{c}$的位移量。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-60.png)

为了简化上式，可以将位移项和样条函数项记为

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-61.png)

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-62.png)

最终得到

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-63.png)

原网格上的每一个顶点$\mathbf{p}_i \in S$都有一个对应的的参数坐标$\mathbf{u}_i = ( u_i,  v_i, w_i)$，且

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-64.png)

每一个顶点都会被施以变换

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-65.png)

因为$\mathbf{d}(\mathbf{u}_i)$中的$N$项是一个常数，可以预计算，所以其效率较高。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-66.png)

通过操纵样条控制点(即指定控制点位移$\mathbf{\delta_c}$)就能够控制模型的变形。和之前提到的方法类似，固定控制区域H(施加位移向量$\mathbf{d}$之后的位置)和固定区域F内的顶点，然后根据给定的$\mathbf{\delta_c}$解下列线性方程

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-67.png)

由于方程左边的矩阵不是方阵，所以可以使用最小二乘法的思想，最小化

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-68.png)

以及控制点的位移量

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-69.png)

不过需要注意两个问题，首先如果上述方程是过定的(Over-Determined)，所以限制条件不能被完美地满足；另外如果是欠定的(Under-Determined)，剩余的自由度是由最小化控制点的位移量决定的，而不是通过光滑算法决定。

样条函数$N$在规则网格(Regular Grid)上如何放置是另外一个问题。如果放置不当，会造成Alias Artifacts(走样/假频)。通过使用更为灵活的且能够更好的表示目标变形的格子框架能够在一定程度上解决这个问题，但是面对复杂的变形的时候，格子框架的选取非常困难。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-70.png)

### Cage-Based Freeform Deformation

Cage-Based Freeform Deformation可以被看作是Lattice-Based Freeform Deformation的一般化情况。Cage-Based Freeform Deformation使用 **控制笼(Control Cage)** 而不是Lattice-Based Freeform Deformation中使用的规则的格子框架。这种控制笼通常是包围着待修改物体的任意三角形网格。相对于格子框架，其能够更好的包裹目标物体。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-71.png)

原始网格$S$上的顶点$\mathbf{p}_i$能够表示为控制笼上顶点的线性组合

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-72.png)

其中权重$\phi$是广义重心坐标(Generalized Barycentric Coordinates)，可以取之前方程中的$N$项。

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-73.png)

顶点的权重$\phi$可以被预计算，这样通过操作控制笼上的顶点

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-74.png)

并使用下面的方法计算逐顶点的位移向量即可

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-75.png)

这样就能比之前使用格子框架能够更为灵活地进行控制。其缺点是由于解是最小范数解(Least Norm Solution)，所以结果并不一定是一个Fair Deformation(不知道怎么翻译)。

## Radial Basis Functions

在Surface-Based Deformations中，通过指定地位移进行插值，并且最小化能量函数得到了相当不错的结果。借用同样的思想，同样地，我们可以对Space Deformation中的函数

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-76.png)

进行插值，并且最小化特定的能量函数。总的来说，我们的目标是找到函数$\mathbf{d}$，能够在位置$\mathbf{p}_i$根据指定的位移向量进行插值运算，且满足给定的限制条件。Radial Basis Functions(RBFs)正是非常适合这类问题的函数。其定义为径向对称核函数$\phi$的叠加，其中心为 $\mathbf{c}_j$，权重为$\mathbf{w}_j$

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-77.png)

其中$\mathbf{\pi}(\mathbf{x})$为保证精度用的多项式项。通常取$\mathbf{c}_j = \mathbf{p}_j$，$\mathbf{w}_j$的值为下列方程的解

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-78.png)

这样RBF函数就能够被应用到顶点上了

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-79.png)

而核函数φ的选择会影响到计算的复杂度和结果的质量。紧支撑径向基函数(Compactly
Supported Radial Basis Functions)会产生稀疏的线性方程，因此其能够被用来对加以很多限制条件的目标进行插值。其缺点在于效果不如全局支持径向基函数(Global Support Radial Basis Functions)。

全局支持径向基函数

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-80.png)

会产生一个三次(??不确定是不是这么翻译的)调和函数(Tri-Harmonic
Function) $\mathbf{d}$

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-81.png)

通过变分法可知，需要最小化的能量函数为

![](/img/post/Polygon-Mesh-Processing-note9-Deformation/img-82.png)

## 线性方法的局限性

{{<figure src = "/img/post/Polygon-Mesh-Processing-note9-Deformation/img-83.png" title = "原始模型">}}

{{<figure src = "/img/post/Polygon-Mesh-Processing-note9-Deformation/img-84.png" title = "使用非线性方法得到的结果(从左往右分别是纯平移变形、弯折变形、扭曲变形)">}}

* Shell-Based Deformation结合Multi-Scale Deformation能在纯平移变形时得到较高质量的结果，且能够保留相当的细节。但是由于Shell Energy是线性的，当对物体施以大尺度旋转变形的时候，结果会非常不理想。

{{<figure src = "/img/post/Polygon-Mesh-Processing-note9-Deformation/img-85.png" title = "Shell-Based Deformation得到的结果">}}

* Gradient-Based Deformation使用对控制点变换的梯度更新每一个面的梯度，因此在对物体施以旋转变形的时候结果会非常理想。但是由于局部旋转的显式传播(Propagation)是对平移敏感的，会导致结果曲面不够光滑，并且会丢失细节特征。

{{<figure src = "/img/post/Polygon-Mesh-Processing-note9-Deformation/img-86.png" title = "Gradient-Based Deformation得到的结果">}}

* Laplacian-Based Deformation隐式地优化了局部旋转，因此相对来说对旋转和平移变形更为友好。但是由于需要对旋转部分线性化，因此在大尺度变形时会导致扭曲。

{{<figure src = "/img/post/Polygon-Mesh-Processing-note9-Deformation/img-87.png" title = "Laplacian-Based Deformation得到的结果">}}
