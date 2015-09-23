---
layout: post
author: mxn
titile: ViewDragHelper解析
category: 技术博文
tag: android
---

2013年谷歌i/o大会上介绍了两个新的layout： SlidingPaneLayout和DrawerLayout，现在这俩个类被广泛的运用，
其实研究他们的源码你会发现这两个类都运用了ViewDragHelper来处理拖动。ViewDragHelper是framework中非常有用的一个工具。

ViewDragHelper解决了android中手势处理过于复杂的问题，在DrawerLayout出现之前，侧滑菜单都是由第三方开源代码实现的，其中著名的当属MenuDrawer，
和slidingMenu.MenuDrawer重写onTouchEvent方法来实现侧滑效果，代码量很大，实现逻辑也需要很大的耐心才能看懂。如果每个开发人员都从这么原始的步奏开始做起，
那对于安卓生态是相当不利的。所以说ViewDragHelper等的出现反映了安卓开发框架已经开始向成熟的方向迈进。

其实ViewDragHelper并不是第一个用于分析手势处理的类，gesturedetector也是，但是在和拖动相关的手势分析方面gesturedetector只能说是勉为其难。

<!-- more -->

## 使用示例

首先我们通过一个简单的例子来看看其快捷的用法

1.自定义ViewGroup




