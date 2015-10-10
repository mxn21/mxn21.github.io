---
layout: post
author: mxn
titile: LayoutAnimation使用
category: 技术博文
tag: android
---

在Android中，最简单的动画就是补间动画了。通过补间动画，可以对一个控件进行位移、缩放、旋转、改变透明度等动画。
但是补间动画只能对一个控件使用，如果要对某一组控件播放一样的动画的话，可以考虑layout-animation。

layout-animation可由xml和代码两种方式配置：

#### XML配置

由于layout-animation是对于某一组控件的操作，就需要一个基本的动画来定义单个控件的动画。另外还可以定义动画的显示顺序和延迟：

    {% highlight xml %}

     <layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
            android:delay="30%"
            android:animationOrder="reverse"
            android:animation="@anim/slide_right" />

    {% endhighlight %}

其中

* android:delay表示动画播放的延时，既可以是百分比，也可以是float小数。
* android:animationOrder表示动画的播放顺序，有三个取值normal(顺序)、reverse(反序)、random(随机)。
* android:animation指向了子控件所要播放的动画。

上述步骤完成之后，就可以将layout-animation应用到ViewGroup中，xml布局添加下面一行就ok：

    {% highlight xml %}
android:layoutAnimation="@anim/list_anim_layout"

    {% endhighlight %}

这样在加载布局的时候就会自动播放layout-animtion。


#### 代码配置

如果在xml中文件已经写好LayoutAnimation，可以使用AnimationUtils直接加载：


    {% highlight java %}

 AnimationUtils.loadLayoutAnimation(context, id)

    {% endhighlight %}

另外还可以手动java代码编写，如：

