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

注意：调用View的scrollTo()和scrollBy()是用于滑动View中的内容，而不是把某个View的位置进行改变。如果想改变莫个View在屏幕中的位置，可以使用如下的方法。
调用public void offsetLeftAndRight(int offset)用于左右移动方法或public void offsetTopAndBottom(int offset)用于上下移动。
如：button.offsetLeftAndRignt(300)表示将button控件向左移动300个像素。






