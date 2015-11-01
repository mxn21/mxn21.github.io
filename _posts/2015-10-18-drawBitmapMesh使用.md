---
layout: post
author: mxn
titile: drawBitmapMesh使用
category: 技术博文
tag: android
---

Canvas提供了一个方法
drawBitmapMesh(Bitmap bitmap, int meshWidth, int meshHeight, float[] verts, int vertOffset, int[] colors,int colorffset,Paint paint)

这个方法可以对bitmap进行扭曲
参数说明如下:
1.bitmap　　　　　需要扭曲的源位图
2.meshWidth 　　控制在横向上把该源位图划成成多少格
3.meshHeight 　　控制在纵向上把该源位图划成成多少格
4.verts 　　　　　长度为(meshWidth + 1) * (meshHeight + 1) * 2的数组，它记录了扭曲后的位图各顶点位置
5.vertOffset         控制verts数组中从第几个数组元素开始才对bitmap进行扭曲

Mesh代表的是网格，这个东西的原理就是按照网格来重新拉伸你的图像,假想在一张图片上有很多网格，如下图。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img121.png)

在这张图上，每一条横线和纵线有一个焦点，我们可以控制这些焦点的位置来重新改变图片的形状和画面，也就是网格怎么扭动，
图像就会怎么动。比如把焦点位置改变成下面这样，图像就跟着扭曲了。


待续

