---
layout: post
author: mxn
titile: SurfaceView浅析
category: 技术博文
tag: android
---

### Surface

Surface对应了一块屏幕缓冲区，每个window对应一个Surface，任何View都要画在Surface的Canvas上,传统的view共享一块屏幕缓冲区，
所有的绘制必须在UI线程中进行。

1、通过Surface（因为Surface是句柄）就可以获得原生缓冲器以及其中的内容.
2、原始缓冲区（a raw buffer）是用于保存当前窗口的像素数据的.
3、Surface中有一个Canvas成员，专门用于画图的.

可以认为Android中的Surface就是一个用来画图形（graphics）或图像（image）的地方。





