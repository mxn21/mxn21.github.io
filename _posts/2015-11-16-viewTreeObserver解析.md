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



