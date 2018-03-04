---
title: "Guide to Computational Geometry Processing Foundations 阅读笔记(1) 多边形网格"
date: 2018-03-03T13:14:20+08:00
draft: false
tags: ["Guide to Computational Geometry Processing Foundations", "阅读笔记"]
---

这本书的前4章讲的是纯数学(看的我一脸懵逼). 所以仔细详细还是从"正体"部分开始看吧, 如果中间碰到问题再往回翻吧. 感觉这书也是硬骨头, , 不知道什么时候才能填完这个坑, 先立个Flag吧.

<!--more-->

# 多边形以及三角形网格基础

对于一个多边形网格实体:

- 可以说是 面$F$的集合, 边$E$的集合以及点$V$的集合. 其中准确地说$E$和$V$是面$F$上的边和点.
- 顶点$i$在空间中对应点的位置记为$\mathbf{p}_i$.
- 全部顶点的个数记为$\|V\|$, 对于边和面同理.
- 记与顶点$i$相邻顶点的集合为$N_i$, 即对于顶点$i$, 满足边$ij$是三角形网格上的边这一条件的顶点$j$组成的集合.
- 与顶点$i$相邻顶点的数目记作$\|N_i\|$, 称作顶点i的 **顶点价(Valence)**.
- 一个顶点的"*One-Ring*"是共享这个顶点多边形的集合, 与改顶点相邻的顶点也被称为是 *One-Ring* 相邻的(如下图所示).

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-0.PNG" title = "顶点以及它的one-ring">}}

很多算法都要求网格是一个符合直觉观念的曲面, 说准确一点就是要求网格表示的是一个 **流形(Manifold)** .

换句话说就是, 曲面任意足够小的区域都存在一个到单位圆盘(Unit Disk)上的可逆映射(即该映射是One-to-one并且是Onto的). 两个三角形的公共边上的一个点就满足这个条件, 而三个三角形的公共边则不满足这个条件, 另外两个只有一个公共点的三角形也不满足这个条件.

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-1.PNG" title = "不是2维流形的两个三角形网格">}}

三角形网格是流形的 **条件** 为: 面只能交与边或者点, 边只能被一个或者两个面共享(取决这条边是处于内部还是处于边界上); 一个点周围的面只能是单循环(Single Cycle)的.

上述条件成立的前提是, 网格不能够出现自相交, 否则这会违反"面只能交与边或者点"这个条件. 其它的不合法的情形如两个只有一条边连接的立方体或者两个只有尖尖部分相连接的椎体.

# 多边形模型的来源以及问题

多边形模型数据的来源主要有以下三种形式

- 2维空间上的散点. 这些通常来源于采集的地形数据, 这些数据拥有的特征能够使其能够被投影到2维空间上, 并且在平面上被三角剖分(通常使用Delaunay三角剖分). 这些数据都能够被看作是对高度函数$f:R^2 \rightarrow R$进行的采样.
- 3维空间上的三点. 这些通常指哪些复杂的3维曲面, 这些曲面因为其拓扑结构未知, 不能很容易的拍平为平面曲面. 常见的作法是将其转换为下列的隐式表示的形式.
- 隐式曲面. 对于等值$\tau$以及函数$\Phi:R^3 \rightarrow R$, 可以得到曲面$S=\Phi^{-1}(\tau)$. 在2维空间上, 等值$\tau$以及$\Phi$对应的是一条闭合曲线, 而在3维空间上对应的是一个Watertight的曲面(不透水的闭合曲面). 另一个隐式曲面的来源诸如CT或者是MRI(核磁共振)之类的体积数据.

对于产生的多边形模型, 我们一般将其视为 *离散的集合信号* . 这些网格的连接性以及拓扑信息都有可能会受到信号噪声的影响, 归结起来有下面几点:

- 过采样(Oversampling): 由于采样的时候并没有区分物体的集合细节度的高低, 这就会导致生成的几何模型带有了很多不必要的数据.
- 采样不足(Undersampling): 由于同样的原因, 对一些小尺度的几何特征和尖锐的边角的采样会存在不足, 这回导致最终得到的网格中会存在锯齿.
- 非正则(Irregularity): 对于完全正则的三角形网格, 所以顶点的顶点价(Valence)都是6, 但是只有圆环或者平面是完全正则的. 然而在高度正则化的网格上, 顶点价(Valence)不为6的顶点数目较少. 因为很多的算法对高度正则化的网格有着较好的亲和性, 所以改善网格的正则性是必不可少的. 一般将一些不太好的三角形分为Needles(针头)和Caps(帽子)两类. 前者指存在一个非常非常短的边的三角形, 后者指的是存在一个接近180°内角的三角形.
- 拓扑问题(Topological Issues): 受扫描仪器的限制, 得到的网格往往是一块一块的, 并不是一个整体. 极端情况下还有可能出现独立存在的三角形. 最终得到网格可能是一个流行, 但是其亏格数(Genus)可能多于预期.

# 对多边形网格的操作

## Edge Collapse

如图所示, 该操作移除了一条边以及和这条边相邻的两个面. 因为Edge Collapse能够减小网格的复杂度, 所以常被用在网格简化算法中(如QSlim算法).

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-2.PNG" title = "Edge Collapse以及它的逆操作Vertex Split">}}

Edge Collapse可以看作将两个顶点焊接在一起，并且两个顶点并不一定要是相连的. 如果允许焊接两个不相连的顶点, 就有可能会导致非流形(Non-Manifold)网格的出现. 因为对于被焊接在一起后的新顶点来说, 它存在两个不相连接的one-ring. 如果希望避免这种情况出现, 就应该限制Edge Collapse操作只能对相连接的顶点执行, 并进行下面的测试:

1. 找到与被Collapse的边的两个端点相邻的所有顶点. 如果与该边相邻的面是三角形, 不在该边上的角将会被两个端点连在一起, 这样就是合法的, 因为Collapse后这两个三角形就会消失. 而其它所有的一个顶点和待Collapse边的两个端点相邻的情况(非三角形网格)都是非法的, 因为这会导致一个顶点和焊接后的新顶点之间存在两条边.

2. 如果所有与待Collapse边相邻的面都是三角形, 且边的端点的顶点价(Valence)为3, 则该物体是一个四面体, Collapse操作会使四面体退化.

3. 如果所有与待Collapse边相邻的面都是三角形, 它们在操作后会消失, 且三角形上除这条边上的两个端点外的另一个顶点的顶点价(Valence)会减少1.

4. 对于两个连接两个边界环(Boundary Loops)的边, 一般是不允许进行Collapse操作的, 因为这会构造出一个8字形.

---

其中第三条的原文是

>If a face adjacent to the edge being collapsed is a triangle, it will disappear, and
the valence of the final vertex belonging to this edge will be reduced by one.
If the valence of this vertex is three it will be reduced to two. If the remaining
two faces in the 1-ring of this vertex are triangles, they will become coplanar. If
the faces are not triangles we may wish to allow the collapse since valence two
vertices could be considered legal.

这个地方我想了好久, 感觉莫名奇妙, 为什么Collapse后余下的两个面如果是三角形它们就共面? 不是三角形则可以操作？ 且顶点价(Valence)等于2合法对应的前提条件是什么? 这里都没有说清楚.

---

## Vertex Split

Vertex Split是Edge Collapse的逆操作, 后者降低复杂度, 而前者则带来更多的几何细节.

## Edge Flip

Edge Flip操作移除掉一条边后, 在产生的四边形的对角上添加一条新的边. 通过该操作能够显著的改善网格的正则性(Regularity). 需要注意的是, 某些情况下执行Edge Flip操作会导致非流形(Non-Manifold)网格的产生.

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-3.PNG" title = "">}}

## Edge Split

Edge Split操作在边上插入一个顶点, 并且在该顶点和与该边相邻的两个三角形的与该边相对的顶点之间插入两条新的边.

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-4.PNG" title = "">}}

## Face Split

Face Split操作在三角形内插入一个顶点, 并且在该顶点和三角形三个顶点之间插入三条边.

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-5.PNG" title = "">}}

# 多边形网格的表示

多边形网格最简单的表示方法即使直接存储每一个顶点的位置, 但是点与点之间的连接性确无从知晓, 这样我们就无法根据存储的信息直接对网格进行操作.

一个更为使用且简单的方法是将网格的信息用顶点集合和面索引集合来表示. 对于每一个顶点, 在顶点集合中存储它的相关集合属性. 然后在面索引集合中用索引来引用顶点集合中的元素来表述网格中的每一个面.

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-6.PNG" title = "">}}

但是上述的表示方法还是存在一些问题, 如果我们需要知道某两个的公共边或者某一个顶点的相邻的顶点, 则需要去遍历所有的面.

为了能够快速的查询这些信息, 一种方法是创建其它的数组去存储顶点的相邻信息(如对于某一个顶点, 所有包含该顶点的面). 另外一种方法是通过存储网格中的边来间接地存储相邻信息, 但是这也意味着当网格被修改的时候我们需要去维护这些辅助地数据结构.

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-7.PNG" title = "">}}

上图展示了翼边数据结构(Winged Edge Data Structure), 对于每一个边, 存储了其在顺时针和逆时针方向上的前一个边和后一个边, 该边所相邻的两个面和两个顶点.

注意到当我们想要从一个边前进到其下一个边的时候, 需要考虑是按照顺时针的方向还是按照逆时针的方向, 这使得在遍历某一个面(Circulate a face)的时候需要插入条件判断语句(If...Else...), 在现今的处理器架构中, 如果对于分支的预测失败, 则已经执行的命令将会被丢弃, 这在一定程度上会影响到程序的性能. 正是由于这些原因, 翼边数据结构已经被半边数据结构(Halfedge Data Structure)所取代.

# 半边数据结构(Halfedge Data Structure)

半边数据结构用两条半边来表示一条实际的边, 每一个半边对应了与该边相邻的一个面, 这样在对半边进行遍历的时候顺序就是唯一的了. 同样地, 使用这种数据结构也能够很方便地遍历一个顶点周围的所有的边, 给定一个顶点$v$:

1. 通过顶点$v$, 可以得到从$v$出发的半边的指针$h_0$
2. 通过$h_0$可以得到半边$h_0$指向的顶点$w$($w$与$v$相邻)
3. 通过和$h_0$方向相反的(Opposite)的半边$h_0'$, 可以得到从顶点$v$出发, 指向下一个与$v$相邻顶点的半边$h_1$
4. 反复执行步骤2到步骤3直到回到最初的半边$h_0$

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-8.PNG" title = "通过一条半边可以得到与它方向相反的另一条半边, 其指向的顶点, 其上一条和下一条半边以及半边所处的面.">}}

在一些几何库(如OpenMesh, CGAL等)中会提供一种特殊的叫Circulators遍历方式, 通过这种遍历方式能够按照顺序遍历一个面的所有半边或者从某个顶点出发的所有半边.

# Quad-Edge Data Structure

Quad-Edge Data Structure只能够被用来表示流形(Manifold), Quad-Edge Data Structure中有顶点, 边和面, 其中边是Quad-Edge Data Structure中最重要的, 它存储了全部的拓扑信息. 而顶点和面存储的都是多余信息.

通过将边旋转90度, 用顶点代替面, 用面代替顶点, 我们能够得到原网格的对偶(Dual)网格, 如立方体的对偶是八面体, 而四面体的对偶是另一个四面体.

所谓Quad-Edge, 指的是边自身, 它的对称(取反方向), 它的对偶, 它的对偶的对称:

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-11.PNG" title = "">}}

通过每一条边我们能够索引到下面的信息:

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-9.PNG" title = "">}}

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-10.PNG" title = "">}}

Quad-Edge Data Structure通过存储四倍多余的信息, 在表示和处理涉及到对偶结构的2维流形(如Voronoi图和Delaunay三角剖分)时十分方便.

对于Quad-Edge Data Structure, 只需要下面两种基本的拓扑操作就能完成全部的创建(Construction)和修改(Modification)需求:

{{<figure src = "/img/post/Guide-To-Computational-Geometry-Processing-Foundations-note1-polygon-meshes/img-12.PNG" title = "">}}
