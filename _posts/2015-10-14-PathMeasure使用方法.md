---
layout: post
author: mxn
titile: PathMeasure使用方法
category: 技术博文
tag: android
---

沿预定路径动画图像（无论是直线或曲线）是做动画时一种常见的情况。传统方法是寻找path的函数，带入x的值来计算y的值，
然后使用坐标(x,y)来完成动画。我们可以在一个更简单的方法，就说通过使用path和pathmeasure。

PathMeasure主要是配合Path，来计算Path里面点的坐标的，或者是给一个范围，来截取Path其中的一部分的。






