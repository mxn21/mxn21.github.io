---
layout: post
author: mxn
titile: android中的static关键字
category: 技术博文
tag: android
---



### 在android中不要用static声明view

在android中不要用staitc声明view（包括view的子类）,因为在创建视图的时候会使用LayoutInflater.inflate()来导入视图，
setContentView(R.layout.ID)底层同样是使用inflate导入视图，这需要使用一个当前的Context，并且传递给被导入的视图。

