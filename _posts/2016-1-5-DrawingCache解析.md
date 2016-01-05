---
layout: post
author: mxn
titile: DrawingCache解析
category: 技术博文
tag: android
---

在Android的View的设计中有一个cache的概念，这个cache就是一个bitmap对象。利用这个功能可以对整个屏幕视图进行截屏并生成Bitmap，也可以
获得指定的view的Bitmap对象。
