---
layout: post
author: mxn
titile: ViewTreeObserver解析
category: 技术博文
tag: android
---

ViewTreeObserver是view事件的观察者。这个观察者用来监听视图树，会监听视图树发生全局变化时发出的通知。这里指的全局
事件包括而且不局限在以下几个：整个视图树的布局变化，开始绘制视图，触摸模式改变等等。

ViewTreeObserver是不能被应用程序实例化的，因为它是由视图提供的，通过view.getViewTreeObserver()获取。

ViewTreeObserver提供了view的很多种监听，每一种监听在ViewTreeObserver中都有一个内部类接口来定义。查看源码可以看到
一共有11个内部类，全部保存在CopyOnWriteArrayList数组中，通过ViewTreeObserver.addXXXListener()来添加这些监听，
所以这11个接口对应了11个add方法，但是有一些方法是还没有开放的，用@hide隐藏了。我目前测试的源码在compileSdkVersion 22的基础上。


![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img128.png)




