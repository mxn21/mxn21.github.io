---
layout: post
author: mxn
titile: ImageView scaleType属性分析
category: 技术博文
tag: android
---

scaleType的属性值有：matrix   fitXY  fitStart   fitCenter  fitEnd  center   centerCrop  centerInside

它们之间的区别如下：


matrix 用矩阵来绘制(从左上角起始的矩阵区域)

fitXY  把图片不按比例扩大/缩小到View的大小显示（确保图片会完整显示，并充满View）

fitStart  把图片按比例扩大/缩小到View的宽度，显示在View的上部分位置（图片会完整显示）

fitCenter  把图片按比例扩大/缩小到View的宽度，居中显示（图片会完整显示）

fitEnd   把图片按比例扩大/缩小到View的宽度，显示在View的下部分位置（图片会完整显示）

center  按图片的原来size居中显示，当图片宽超过View的宽，则截取图片的居中部分显示，当图片宽小于View的宽，则图片居中显示

centerCrop  按比例扩大/缩小图片的size居中显示，使得图片的高等于View的高，使得图片宽等于或大于View的宽;或者使得图片的宽等于View的宽，使得图片高等于或大于View的高

centerInside  将图片的内容完整居中显示，使得图片按比例缩小或原来的大小（图片比View小时）使得图片宽等于或小于View的宽 （图片会完整显示）

<!-- more -->

附上两张实验的截图:

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img22.png)

图1：　图片比ImageView大的截图

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img23.png)

图2： 图比ImageView小 实验截图


总结：scaleType的属性值可以分为几类：

第一类：matrix ，center不会改变原图大小。放在固定宽高的imgview中，要么图片小，留下一大片空白，要么图片太大显示不完。
这两个区别就在于如果图片太大，matrix显示的是左上角，center显示的是图片中间。图片大小比例都不变。

第二类：fitStart ， fitEnd ， fitCenter，这3个效果差不多，都是适应长的边，适应其中一条边，另一条边小于边缘。图片比例不变。

第三类：fitXY。这个是最简单的。适应两条边 。图片大小比例都变 。

第四类：centerCrop 都是适应短的边，适应其中一条边，另一条边大于边缘。图片比例不变。

第五类：centerInside