---
layout: post
author: mxn
titile: setWillNotDraw和setDescendantFocusability详解
category: 技术博文
tag: android
---

### setWillNotDraw(false)

如果要自定义一个ViewGroup，重写onDraw方法，它是不会被调用的，下面分析一下原因和解决方法。

先重写一个ViewGroup

    {% highlight java  %}

<com.test.demo.MyLinearLayout xmlns:android="http://schemas.android.com/apk/res/android"

    android:id="@+id/ll_absolute"

    android:orientation="vertical"

    android:layout_width="fill_parent"

    android:layout_height="fill_parent"

    android:background="#FF000000">

</com.test.demo.MyLinearLayout>

    {% endhighlight %}
大概的架构是，MyLinearLayout从LinearLayout派生出来，然后在程序中重载OnDraw(Canvas canvas)。
但是，onDraw不会被调用。我们可能会遇到这个问题：如果不给LinearLayout设置一个背景，系统是不会调用onDraw时，
也就是说，我们重写的onDraw是不会调用的。当设置一个背景后，onDraw就会被调用。

造成这种现象的原因是继承自LinearLayout，而LinearLayout这是一个容器，ViewGroup它本身并没有任何可画的东西，
它是一个透明的控件，因些并不会触发onDraw，但是你现在给LinearLayout设置一个背景色，
其实这个背景色不管你设置成什么颜色，系统会认为，这个LinearLayout上面有东西可画了，因此会调用onDraw方法。

我们可以仔细分析View的源码，它有一个方法View#draw(Canvas)方法，这里面有两个地方调用onDraw，它的条件都是：
if (!dirtyOpaque) onDraw(canvas);

也就是说，如果dirtyOpaque是true的话，onDraw就不会调用，而dirtyOpaque的值的计算代码如下：

    {% highlight java  %}
final boolean dirtyOpaque = (privateFlags & DIRTY_MASK) == DIRTY_OPAQUE &&
                (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
    {% endhighlight %}

<!-- more -->

当View初始他时，它会调用一个私有方法：computeOpaqueFlags，这里面列出了不透明的三个条件：
// Opaque if:
//   - Has a background
//   - Background is opaque
//   - Doesn't have scrollbars or scrollbars are inside overlay

View还提供了一个重要的方法：setWillNotDraw，我们看一看它的实现：

    {% highlight java  %}
 public void setWillNotDraw(boolean willNotDraw) {
     setFlags(willNotDraw ? WILL_NOT_DRAW : 0, DRAW_MASK);

 }

     {% endhighlight %}