---
layout: post
author: mxn
titile: clipChildren属性分析
category: 技术博文
tag: android
---

1.clipChild用来定义他的子控件是否要在他应有的边界内进行绘制。 默认情况下，clipChild被设置为true。 也就是不允许进行扩展绘制。

2.clipToPadding用来定义ViewGroup是否允许在padding中绘制。默认情况下，cliptopadding被设置为ture， 也就是把padding中的值都进行裁切了。

这两个属性联合起来能干什么呢？用来做一些类似于点击放大等点击特效非常合适.不用去更改布局，只需加入这两个属相，并引入动画效果就完成了。

ClipChildren属性对需要做动画的View非常有用，特别是对ScaleAnimation。
需要注意的是ClipChildren ＝ false只是告诉他的子View可以超出他本身的大小，并不是说他自己可以超出他的父View的大小，
所以如果你想允许某个View超过其父View的边界，你需要一直找到最顶层的父View并设置ClipChildren = false。

下面需要提一下的是一个关于ClipChildren的坑，在4.2的系统上(4.3未测试，4.4及5.0版本没有这个 BUG)，在启用了hardware accelerated的情况下，
ScaleAnimation会出现撕裂等不流畅的现象，处理方法如下：

    {% highlight java %}
if (Build.VERSION.SDK_INT < Build.VERSION_CODES.KITKAT) {
    parentView.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
 }
    {% endhighlight %}

这里的parentView一定要是你启用了ClipChildren = false的那个ViewGroup，否则没有效果，最佳实践是只在需要做ScaleAnimation的时候才关闭硬件加速，做完动画后应该重新开启。

看看最后的效果吧， 点击第三个机器人就会播放一个变大的效果（类似于心变大的效果）

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img114.png)



