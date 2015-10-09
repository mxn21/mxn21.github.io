---
layout: post
author: mxn
title: android listview和gridview属性积累
category: 技术博文
tag: [android]
---

## ListView

#### stackFromBottom属性

这只该属性之后你做好的列表就会显示你列表的最下面，值为true和false。这个属性并不能是listview倒序排列
![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img11.gif)

#### transciptMode属性

需要用ListView或者其它显示大量Items的控件实时跟踪或者查看信息，并且希望最新的条目可以自动滚动到可视范围内。通过设置的控件transcriptMode属性可以将Android平台的控件（支持ScrollBar）自动滑动到最底部。
 android:transcriptMode="alwaysScroll"    

#### clipToPadding属性

设置listview距离上面的控件的padding,并且允许在padding中滑动。
普遍的情况下,对于任何具有这个属性的view,例如listview,clipToPadding就是说控件的绘制区域是否在padding里面的，true的情况下如果你设置了padding
那么绘制的区域就往里缩.如果为false，则绘制的区域不会变小，会绘制在padding上。
android:paddingTop="10dp"
android:clipToPadding  = "false"

## gridView

#### stretchMode 

gridview的item两边显示多余的空白部分，设置 android:stretchMode="none"属性即可。

#### clickable 

gridview调用setonitemclicklistener时确保item的clickable属性为false

#### scrollbarStyle

当设置paddingright属性时，滚动条显示在里边，要想滚动条显示在外边，设置android:scrollbarStyle=" outsideOverlay "属性即可

