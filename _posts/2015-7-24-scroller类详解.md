---
layout: post
author: mxn
titile: scroller类详解
category: 技术博文
tag: android
---

我们知道想把一个View偏移至指定坐标(x,y)处，利用scrollTo()方法直接调用就OK了，但我们不能忽视的是，该方法本身
来的的副作用：非常迅速的将View/ViewGroup偏移至目标点，而没有对这个偏移过程有任何控制，对用户而言可能是不太
友好的。于是，基于这种偏移控制，Scroller类被设计出来了，该类的主要作用是为偏移过程制定一定的控制流程，
从而使偏移更流畅，更完美。

<!-- more -->




