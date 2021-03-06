---
title: "Polygon Mesh Processing 阅读笔记(3) 微分几何"
date: 2018-02-28T18:29:17+08:00
draft: false
tags: ["Polygon Mesh Processing", "阅读笔记"]
---
<!--more-->

## 曲线

曲线是二维空间上可微分的一维流形。曲线可以用参数方程表示为如下形式：

$$ \mathbf{p}(u) =
\begin{pmatrix}
x(u)\\\\\
y(u)\\\\\
\end{pmatrix},
u \in[a, b] \subset R $$

其中$x$和$y$分别是关于$u$的可微函数，那么曲线在某一点的切向量则为各分量的一阶导数组成的向量，即：

$$ \mathbf{p'}(u) =
\begin{pmatrix}
x'(u)\\\\\
y'(u)\\\\\
\end{pmatrix},
u \in[a, b] \subset R $$

借由上式，如果$p'(u) $在$ u $处不为0，则把这一点成为曲线的 **正则点** 。曲线上的点处处正则的曲线称为 **正则曲线** (Regular Curve)。

下式可以求曲线在某一点的法向量的值：

$$
\mathbf{n}(u) = \frac{\mathbf{p'}(u)^{\perp}}{\Vert \mathbf{p'}(u)^{\perp} \Vert}
$$

同样的曲线是可以通过参数变换使用不同的参数来表示的。曲线的微分几何关注诸如 **弧长** 、**曲率** 之类的，独立于特定参数之外的属性，也就是说无论参数如何变换，这些属性的值都是相等的。

### 弧长

对于上述曲线，起始点$a$到曲线上任意点$u$之间的弧长可以表示为：

$$
s(u) = \int_{a}^{u} \Vert \mathbf{p'}(s) \Vert du, u \in [a, b]
$$

即弧长是切向量长度对曲线参数的积分。可以发现，**弧长** 独立与特定参数，并且将参数$u$从区间$[a, b]$映射到了区间$[0, L]$，其中$L$是曲线的弧长。

可以发现这是一个变上限积分函数，对两侧同时求导得到：

$$
\frac{ds}{du} = \Vert \mathbf{p'}(u) \\Vert
$$

当切向量的模为1，即曲线的切向量为单位向量场的时候，参数u就是曲线的弧长参数了

### 曲率

曲线的 **曲率** (curvature)就是针对曲线上某个点的切线方向角对弧长的转动率，即单位弧长内曲线转过的角度。

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-0.jpeg)

假设正则曲线的参数方程的参数为其弧长，图像如上图所示，$\alpha$表示是曲线上的切向量(即$p(s)$的导数$p'(s)$)，根据定义曲线的曲率为：

$$
\kappa(s) = \Vert \alpha'(s) \Vert = \lim\limits_{\Delta s \to 0}\|{\frac{\Delta \theta}{\Delta s}\|}
$$

其中$\theta$表示的是$\alpha(s)$和$\alpha(s+\Delta s)$两个向量的夹角，要证明这个式子只需要按照导数定义展开即可：

$$
\kappa(s) = \lim\limits_{\Delta s \to 0} {\frac{\|\alpha(s+\Delta s) - \alpha(s)\|}{\| \Delta s \|}}
$$

$$
\kappa(s) = \lim\limits_{\Delta s \to 0} {\frac{2|\ sin(\frac{\Delta \theta}{2}) \|}{\| \Delta s \|}}
$$

$$
\kappa(s) = \lim\limits_{\Delta s \to 0} {\| \frac{\Delta \theta}{\Delta s} \|}
$$

对于曲率还有另外一个很重要且相关的属性，即 **曲率半径** ，即把那一段曲线尽可能地微分，直到最后近似为一个圆弧，此圆弧所对应的半径即为曲线上该点的曲率半径。这个圆弧所对应的圆一般称作：Osculating Circle(密切圆)。

我们知道弧长与半径的比值是弧度。对于这一段圆弧 **曲率** 的值为弧度于弧长的比值，而半径的值为弧长于弧度的比值。

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-1.jpeg)

对于 **曲率** 和 **曲率半径** ，可以得到下面的关系：

$$
\kappa = \frac{1}{R}
$$

## 曲面

### 曲面的参数化表示

以地球地图的展开为例，地球表面是一个闭合的曲面，为了印刷地图，一般需要将其表面进行展开。

展开之前，首先沿着子午线将其“切开”，然后按下面这个样子进行展开：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-2.jpeg)

可以发现，北极点被变换为了线段$AC$，而南极点被变换了线段$BD$

这样的一个球面，假设半径为$R$，有两种坐标表示的方法，分别是：$(x, y, z)$和$(\theta, \phi)$

前一种非常好理解，即球面某一点在3维笛卡尔坐标系下的坐标，那么球面的可以用下列隐式方程表示：

$$
  x^2+y^2+z^2=R^2
$$

通过该方程能够很快速地判断空间中某个点与球面地位置关系。

后一种坐标中有两个参数$\theta$和$\phi$，其意义可以通过下面这张图来理解(和球坐标非常类似)

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-3.jpeg)

理解后就不难得出两种坐标的转换方法：

$$
\mathbf{x}(\theta, \phi) =
\begin{pmatrix}
x(\theta, \phi)\\\\\
y(\theta, \phi)\\\\\
z(\theta, \phi)\\\\\
\end{pmatrix} =
\begin{pmatrix}
R\ cos(\theta) cos(\phi)\\\\\
R\ sin(\theta) cos(\phi)\\\\\
R\ sin(\phi)\\\\\
\end{pmatrix}
$$

其中$\theta$的取值范围为$[0, 2\pi]$，$\phi$的取值范围为$[-0.5\pi, 0.5\pi]$，可以发现通过这张表示方法将“方形”区域映射到了一个球面上。

通过$\theta$和$\phi$这两个参数，可以画出两组类似经纬度的平行线，通过这些平行线，可以清楚的观察出球面不同部分被扭曲的程度。

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-4.jpeg)

### 度量性质(Metric Property)

假设一个三维曲面的参数方程如下

$$
\mathbf{x}(\theta, \phi) =
\begin{pmatrix}
x(u, v)\\\\\
y(u, v)\\\\\
z(u, v)\\\\\
\end{pmatrix}, (u,v) \in \Omega \subset R^2
$$

其中$x$,$y$,$z$是关于参数$u$,$v$的可微函数，$\Omega$是参数$u$,$v$的定义域。

同曲线类似，曲面的度量是由它的一阶导数决定。$\mathbf{x}$关于参数$u$, $v$的偏导数如下

$$
\mathbf{x}_u(u_0, v_0) := \frac{\partial{\mathbf{x}}}{\partial{u}}(u_0, v_0)
$$

$$
\mathbf{x}_v(u_0, v_0) := \frac{\partial{\mathbf{x}}}{\partial{v}}(u_0, v_0)
$$

这两个偏导数表示的是如下两条曲线上的切向量

$$
\mathbf{C}_u(t)=\mathbf{x}(\mathbf{u_0+t, v_0})
$$

$$
\mathbf{C}_v(t)=\mathbf{x}(\mathbf{u_0, v_0+t})
$$

很明显这两个方程分别是当曲面方程的某个参数固定后，以另一个参数为参数的方程。

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-5.jpeg)

通过上面这张图，能够很清晰的看出$\mathbf{C}_v$, $\mathbf{C}_u$, $\mathbf{x}_v$, $\mathbf{x}_u$的具体含义。

如果想要表示表示在平面某一点的法向量也很简单，曲线方程在某一点关于参数 *u* ， *v* 的偏导数确定了两条切向量$\mathbf{x}_v$, $\mathbf{x}_u$，将这两个向量做叉积即可得到曲面在这一点的法向量

$$
\mathbf{n} = \frac{\mathbf{x}_u \times \mathbf{x}_v}{\Vert \mathbf{x}_u \times \mathbf{x}_v \Vert}
$$

上面的导数方向只有沿两个参数的方向，如果要求曲面关于某一点在任意方向的导数，可以引入 **方向导数** 的概念。

在求解方向导数的时候需要给定一个方向向量，由于曲面方程以参数方程的形式给出，先定义一个在曲线方程参数空间$u$, $v$下的方向向量

$$
\mathbf{\bar{w}}=(u_w, v_w)
$$

那么曲面通过这一点，在参数空间上沿上述方向前进的曲线方程可以表示为

$$
\mathbf{C}_\mathbf{w}(t)=\mathbf{x}(u_0+t u_w, v_0+t v_w)
$$

这时曲面在点$(u_0, v_0)$处，$\mathbf{w}$方向的方向导数为：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-6.jpeg)

向量$\mathbf{w}$是定义在三维空间下的，而已知的方向向量是在二维参数空间上的，现在要将其从参数空间变换为曲面上的切向量：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-7.jpeg)

只需要应用到雅可比矩阵即可完成这个变换：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-8.jpeg)

此时雅可比矩阵的值为：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-9.jpeg)

### 曲面的第一基本型

通过上面求解方向导数的过程可以发现，雅可比矩阵代表了一种从参数的定义域空间到曲面坐标空间的变换。通过雅可比矩阵可以知道一些量，诸如角度、距离和面积等，在这两个空间之间的映射关系。

假设又两个单位向量$\mathbf{w}_1$, $\mathbf{w}_2$，这两个向量之间夹角的余弦值等于两向量的内积。

向量在曲面空间和参数空间下的表示形式不同，单数可以明确的一点是，无论如何表示，向量之间的夹角是不会变的。

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-10.jpeg)

在上面的等式中，$\mathbf{J}$乘以$\mathbf{J}_T$这一部分就被称为 **曲面的第一基本型**

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-11.jpeg)

借由$\mathbf{Ⅰ}$，要通过参数来表示下面曲线的弧长：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-12.jpeg)

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-13.jpeg)

首先观察曲面的弧长公式：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-14.jpeg)

接下来，用参数$t$来表示切向量$\mathbf{w}(u_t, v_t)$，则其模长为：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-15.jpeg)

最后带入计算可以得到弧长公式：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-16.jpeg)

同理，可以用下面的方法求得曲面的面积：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-17.jpeg)

### 曲面的曲率

曲面的曲率的定义是由曲线的曲率的定义扩充而来的，对于曲面上的一点，存在无数个切向量。对于曲面上的一点$\mathbf{p}$，以及一条切向量$\mathbf{t}$，这时可以定义曲率为：切向量$\mathbf{t}$和曲面在这一点的法向量所成平面与曲面相交形成的直线在点$\mathbf{p}$处的曲率。

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-18.jpeg)

将这个曲率写成式子为：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-19.jpeg)

其中$\mathbf{Ⅱ}$为第二基本型

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-20.jpeg)

上面关于曲面曲率的函数在切线方向变化的时候会有两个极值(极大值和极小值)，一般称它们为 **主曲率(principal curvatures)** ，如果两极值不相等，就把取这两个极值时的两个切向量称为 **主方向(principal directions)** 。如果两极值相等，则曲面上这一点称为 **脐点(umbilical)** ，曲面上这一点的所有切向量都可以称为 **主方向(principal directions)** ，并且曲面这一点各方向的曲率相等。特殊地，当且仅当曲面为球面或平面时，其上所有的点都是 **脐点(umbilical)** 。

---

对于曲面的两个 **主曲率** 和其在同一点任意方向的曲率，有如下的关系：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-21.jpeg)

其中$\phi$为 **主方向** $\mathbf{t}_1$和指定方向$\mathbf{t}$的夹角。可以看出，曲面的曲率仅仅由其两个 **主曲率** 决定，这一点任意方向的法曲率都是这两个 **主曲率** 的凸组合(convex combination)，另外还能得出的一点是 **主方向** 永远是相互正交的。

---

曲面的某个区域内的性质同样可以用 **曲率张量** 来表示，**曲率张量** $\mathbf{C}$可以用下面的方法得到：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-22.jpeg)

其中D是对角线元素为$\kappa_1$, $\kappa_2$, $0$的三阶方阵，P也为三阶方阵，由$\mathbf{t_1}$, $\mathbf{t_2}$ , $\mathbf{n}_1$三个列向量组成。

---

另外，还有两种广泛使用的描述曲率的方式：

* **平均曲率**

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-23.jpeg)

*  **高斯曲率**

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-24.jpeg)

 高斯曲率可以将曲面上的点分为3类：

* **椭圆点(elliptical point)**  $K > 0$：椭圆点在其附近的区域上通常是凸出的

* **双曲线点(hyperbolic point)** $K < 0$：双曲点在其附近的区域上通常是马鞍形

* **抛物线点(parabolic point)** $K = 0$：抛物线点通常在椭圆曲线和双曲线区域的分界线处

高斯曲率和平均曲率通常用在曲面的可视化分析上

![左图为平均曲率，右图为高斯曲率](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-25.jpeg)

### 内蕴几何学(Intrinsic geometry)

在微分几何中，那些只依赖于 **第一基本型** 的属性被称为是 **内蕴的(Intrinsic)**。直观上来说它们可以仅仅通过曲面二维特征来导出。例如曲面上曲线的长度，角度等都是 **内蕴的(Intrinsic)**。

对于 **高斯曲率** 和 **平均曲率** ，前者在等距变换下是不变的，所以它是 **内蕴的(Intrinsic)** ，即 **高斯曲率** 是可以由 **第一基本型** 直接决定的；而后者则不是，它依赖于曲面。

**内蕴的(Intrinsic)** 通常被用来取表示参数的独立性。

### 拉普拉斯算子

一般称某函数梯度的散度为 **拉普拉斯算子**，对于二元函数$f(u,v)$，其在欧式空间上的二阶差分算子(拉普拉斯算子)可以写为：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-26.jpeg)

拉普拉斯算子还可以推广到二阶流形曲面$S$上，其推广形式称为 **拉普拉斯-贝尔特拉米算子** ，定义为：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-27.jpeg)

对于曲面上某一个具体的点$\mathbf{x}$，其 **拉普拉斯-贝尔特拉米算子** 和其 **平均曲率** 存在下面的关系：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-28.jpeg)

虽然这个式子说明 **平均曲率** (非内蕴的)和 **拉普拉斯-贝尔特拉米算子** 之间存在某种关系，但是 **拉普拉斯-贝尔特拉米算子** 本身仅取决于 **第一基本型**，是内蕴的。

## 离散微分算子

由于3D网格并不是连续的，而上面的讨论是建立在曲面是光滑的基础之上的。要将上述算子运用到3D网格上，需要将网格看作一个很粗糙的曲面，然后通过网格数据去计算这个近似曲面的微分属性。

### 局部平均区域

一般的想法就是计算网格某个点以及与其相邻点的微分属性的平均值。

当网格某个点以及与其相邻点组成这个区域的面积较大的时候，通过计算平均值得到的微分属性会很稳定；而面积较小的时候，精细的变化则会被更好的保留。

常用的由下面三种定义这个面积的方法，其区别主要是在顶点周围的三角形中取点的方式不同：

{{< figure src="/img/post/Polygon-Mesh-Processing-note3-Geometry/img-29.jpeg" title="重心(左图)，外心(中图)，混合点(右图)" >}}


其中右图中，当三角形为钝角三角形时则取中心点对边的中点，否则取三角形的外心。

### 法向量

在3D网格中，要计算某个三角面的法向量是比较容易的，只需要取两条边向量坐叉乘即可：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-30.jpeg)

如果要计算某个顶点的法向量，同样考虑对顶点周围相邻的三角形的法向量做加权平均：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-31.jpeg)

权值αT的取法，一般常用的有下面几种：

* $\alpha_T$取常数1，这样计算的时候就忽略了邻边的长度，三角形的面积，角度。对于不规则的网格，计算的结果一般都是违反直觉的。

* $\alpha_T$取三角形的面积，这样取的好处是便于计算(只需要进行叉乘运算就可以了，还不用对向量进行单位化)，不过这种方法得到的结果有的时候也会出现违反直觉的情况。

* $\alpha_T$取邻边的夹角，不过由于计算过程涉及到了三角函数，效率上相对要低一些，不过效果比前两者好。

### 梯度

同样是基于加权平均的方法，求解网格中某个三角形上某一点的坐标可以由三个顶点的梯度根据重心坐标的三个权值做加权平均。

对于分段线性函数$f$来说，其在三角形顶点上有对应的值。可以考虑用拉格朗日插值法来表示三角上任意一点的函数值($\mathbf{u}$是二维参数)：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-32.jpeg)

由于拉格朗日插值公式的基函数$B$具有下面的性质

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-33.jpeg)

两边同时做梯度运算可以得到

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-34.jpeg)

消去$B_i$后原来的式子为

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-35.jpeg)

顶点i处基函数的梯度为从顶点$i$处沿着对边上高的方向的向量，且向量的模长为高的倒数，化简后(向量旋转90度后除以底边的长得到单位向量，再除以高度的结果，其中底边长乘以高度整好为面积的两倍)为：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-36.jpeg)

代入后可以得到三角形上分段线性函数的梯度为

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-37.jpeg)

### 离散形式的拉普拉斯算子

* Uniform形式的拉普拉斯算子

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-38.jpeg)

这一种形式直观上来说就是以中心点$i$为起点，相邻顶点平均值为终点的向量。

由于平面的平均曲率$H$为$0$，这时算子的结果应该是$0$，不过上式的结果并不一定是非0的，所以这种方法不太适合用在非等距网格上。

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-28.jpeg)

这种方法只考虑了网格的连接性，所以使用范围有限。

* 余切形式的拉普拉斯算子

这种形式更加的精准，直接计算顶点$v_i$周围的平均区域(之前提到过，有若干种取法)，然后对其梯度的散度进行曲面积分，然后使用 *散度定理(高斯公式)* 进行展开计算，最后可以得到：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-40.jpeg)

* 离散散度

因为拉普拉斯算子的定义为是梯度的散度，对于每一个三角形T给定一个向量$\mathbf{w}$(如个给定分段线性函数$f$下的梯度向量)，则其散度为

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-41.png)

### 离散曲率

根据上面的式子，可以得到在离散形式下的 **平均曲率** ：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-42.jpeg)

书中引用 **Meyer et al. 03** 这篇文章中提到了离散形式下 **高斯曲率** 的表示方式：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-43.jpeg)

根据 **高斯曲率** 、**平均曲率** 和两个 **主曲率** 的关系，可以得到 **主曲率** 的计算方法：

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-44.jpeg)

### 离散形式的曲率张量

![](/img/post/Polygon-Mesh-Processing-note3-Geometry/img-45.jpeg)

其中$\beta(\mathbf{e})$表示和边$\mathbf{e}$相邻三角形所在平面的有方向的二面角，$\mathbf{e} \cap A(v)$表示边$\mathbf{e}$在区域A中的长度，$\mathbf{\bar{e}}$指边$\mathbf{e}$的单位向量。
