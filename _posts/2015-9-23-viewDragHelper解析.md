---
layout: post
author: mxn
titile: ViewDragHelper解析
category: 技术博文
tag: android
---

2013年谷歌i/o大会上介绍了两个新的layout： SlidingPaneLayout和DrawerLayout，现在这俩个类被广泛的运用，
其实研究他们的源码你会发现这两个类都运用了ViewDragHelper来处理拖动。ViewDragHelper是framework中非常有用的一个工具。


