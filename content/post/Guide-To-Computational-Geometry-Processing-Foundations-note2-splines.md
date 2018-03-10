---
title: "Guide to Computational Geometry Processing Foundations 阅读笔记(2) 样条线(Splines)"
date: 2018-03-03T19:30:20+08:00
draft: true
tags: ["Guide to Computational Geometry Processing Foundations", "阅读笔记"]
---
<!--more-->

这一章会讨论在CAD中经常使用的B-Splines, NURBS(Non-Uniform Rational B-Splines). B-Splines(以及其它的参数曲面)相较于细分曲面(Subdivision Surface)的主要缺点是: 曲面的每一个小块(Patch)之间的连续性在进行网格变形(Deformation)时很难维护, 因此细分曲面更适合做动画. 但是细分曲面在小块(Patch)之间的缝隙(Seam)处会出现问题.

# 参数化(Parameterization)
