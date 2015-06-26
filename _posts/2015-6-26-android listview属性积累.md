---
layout: post
author: mxn
title: android listview属性积累
category: 技术博文
tag: [android]
---

#### stackFromBottom属性
这只该属性之后你做好的列表就会显示你列表的最下面，值为true和false。这个属性并不能是listview倒序排列
![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img11.gif)

#### transciptMode属性

需要用ListView或者其它显示大量Items的控件实时跟踪或者查看信息，并且希望最新的条目可以自动滚动到可视范围内。通过设置的控件transcriptMode属性可以将Android平台的控件（支持ScrollBar）自动滑动到最底部。
 android:transcriptMode="alwaysScroll"    

