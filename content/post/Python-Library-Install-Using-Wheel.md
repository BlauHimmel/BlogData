---
title: "在Windows上使用Wheel安装Python库"
date: 2018-04-09T13:50:20+08:00
draft: false
tags: ["Python"]
---

Python的有些库(如Numpy)在Windows上使用pip直接安装的时候需要进行编译, 编译的过程很有可能报出各种各样的错误, 处理起来比较麻烦. 使用Wheel进行安装相对会更简单方便一些.

<!--more-->

首先安装Wheel

```cmd
pip install wheel
```

然后到这里下载对应的whl文件

[Unofficial Windows Binaries for Python Extension Packages](https://www.lfd.uci.edu/~gohlke/pythonlibs/#numpy)

最后使用下面的命令安装下载下来的库文件即可

```cmd
pip install xxxx.whl
```
