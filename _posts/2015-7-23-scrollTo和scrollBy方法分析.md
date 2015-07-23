---
layout: post
author: mxn
titile: scrollTo和scrollBy方法分析
category: 技术博文
tag: android
---

android系统手机屏幕的左上角为坐标系，同时y轴方向与笛卡尔坐标系的y轴方向想反。通过提供的api如getLeft , getTop, getBottom,
getRight可以获得控件在parent中的相对位置。同时，也可以获得控件在屏幕中的绝对位置。

当我们编写一些自定义的滑动控件时，会用到一些api如scrollTo(),scrollBy(),getScrollX(), getScrollY()。
由于常常会对函数getScrollX(), getScrollY()返回的值的含义产生混淆，尤其是正负关系。本文将讲解他们的用法。



