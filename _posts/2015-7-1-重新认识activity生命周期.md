---
layout: post
author: mxn
titile: 重新认识activity生命周期
category: 技术博文
tag: android
---

重新认识activity生命周期

#### onWindowFocusChanged重要作用

Activity生命周期中，onStart, onResume, onCreate都不是真正visible的时间点，
真正的visible时间点是onWindowFocusChanged()函数被执行时。从onWindowFocusChanged被执行起，用户可以与应用进行交互了，
而这之前，对用户的操作需要做一点限制。

这个onWindowFocusChanged指的是这个Activity得到或者失去焦点的时候 就会被调用
也就是说 如果你想要做一个Activity一加载完毕，就触发什么的话 完全可以用这个。

使用一个view的getWidth() getHeight() 方法来获取该view的宽和高，返回的值却为0。
如果这个view的长宽很确定不为0的话，那很可能是你过早的调用这些方法，也就是说在这个view被加入到rootview之前你就调用了这些方法，返回的值自然为0.
解决该问题的方法有很多，主要就是延后调用这些方法。可以试着在onWindowFocusChanged()里面调用这些方法,验证时可以获取到View的宽高的。




