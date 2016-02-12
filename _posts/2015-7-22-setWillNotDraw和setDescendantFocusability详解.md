---
layout: post
author: mxn
titile: setWillNotDraw和setDescendantFocusability详解
category: 技术博文
tag: android
---

## setWillNotDraw(false)

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
    /**
      * If this view doesn't do any drawing on its own, set this flag to
      * allow further optimizations. By default, this flag is not set on
      * View, but could be set on some View subclasses such as ViewGroup.
      *
      * Typically, if you override {@link #onDraw} you should clear this flag.
      *
      * @param willNotDraw whether or not this View draw on its own
      */
 public void setWillNotDraw(boolean willNotDraw) {
     setFlags(willNotDraw ? WILL_NOT_DRAW : 0, DRAW_MASK);
 }
     {% endhighlight %}

从这个方法的注释，我们可以看出，如果你想重写onDraw的话，你应该调用这个方法来清除flag，
所以如果我们想要重写LinearLayout的onDraw的话，我们也可以在其构造方法中调用setWillNotDraw方法。
在ViewGroup初始他时，它调用了一个私有方法：initViewGroup，它里面会有一句setFlags(WILL_NOT_DRAW, DRAW_MASK);
相当于调用了setWillNotDraw(true)，所以说，对于ViewGroup，它就认为是透明的了。如果我们想要重写onDraw，就需要调用setWillNotDraw(false)


### 总结一下

1.ViewGroup默认情况下，会被设置成WILL_NOT_DRAW，这是从性能考虑，这样一来，onDraw就不会被调用了。

2.如果我们要重要一个ViweGroup的onDraw方法，有两种方法：
在构造函数里面，给其设置一个颜色，如#00000000。
在构造函数里面，调用setWillNotDraw(false)，去掉其WILL_NOT_DRAW flag。


## setDescendantFocusability函数


这个函数是在ViewGroup里定义的，主要用于控制child View获取焦点的能力，比如是否阻止child View获取焦点。

他有三个常量可供设置

1.FOCUS_BEFORE_DESCENDANTS ViewGroup本身先对焦点进行处理，如果没有处理则分发给child View进行处理

2.FOCUS_AFTER_DESCENDANTS 先分发给Child View进行处理，如果所有的Child View都没有处理，则自己再处理

3.FOCUS_BLOCK_DESCENDANTS ViewGroup本身进行处理，不管是否处理成功，都不会分发给ChildView进行处理

我们看下这个方法的实现
Setdescendantfocusability(int focusability)代码

    {% highlight java  %}
public void setDescendantFocusability(int focusability) {
        switch (focusability) {
            case FOCUS_BEFORE_DESCENDANTS:
            case FOCUS_AFTER_DESCENDANTS:
            case FOCUS_BLOCK_DESCENDANTS:
                break;
            default:
                throw new IllegalArgumentException("must be one of FOCUS_BEFORE_DESCENDANTS, "
                        + "FOCUS_AFTER_DESCENDANTS, FOCUS_BLOCK_DESCENDANTS");
        }
        mGroupFlags &= ~FLAG_MASK_FOCUSABILITY;
        mGroupFlags |= (focusability & FLAG_MASK_FOCUSABILITY);
    }
     {% endhighlight %}

可以看到，只有这三个常量可以设置，不是这三个常量会抛出异常的。
设置后，会在requestFocus(int direction, Rect previouslyFocusedRect) 方法里根据设置进行相应的处理。来看下实现

Requestfocus(int direction, rect previouslyfocusedrect)代码

    {% highlight java  %}
public boolean requestFocus(int direction, Rect previouslyFocusedRect) {
        if (DBG) {
            System.out.println(this + " ViewGroup.requestFocus direction="
                    + direction);
        }
        int descendantFocusability = getDescendantFocusability();
        switch (descendantFocusability) {
            case FOCUS_BLOCK_DESCENDANTS:
                return super.requestFocus(direction, previouslyFocusedRect);
            case FOCUS_BEFORE_DESCENDANTS: {
                final boolean took = super.requestFocus(direction, previouslyFocusedRect);
                return took ? took : onRequestFocusInDescendants(direction, previouslyFocusedRect);
            }
            case FOCUS_AFTER_DESCENDANTS: {
                final boolean took = onRequestFocusInDescendants(direction, previouslyFocusedRect);
                return took ? took : super.requestFocus(direction, previouslyFocusedRect);
            }
            default:
                throw new IllegalStateException("descendant focusability must be "
                        + "one of FOCUS_BEFORE_DESCENDANTS, FOCUS_AFTER_DESCENDANTS, FOCUS_BLOCK_DESCENDANTS "
                        + "but is " + descendantFocusability);
        }
    }
       {% endhighlight %}

通过这里的实现可以看到上面定义的三个常量设置的意思。

所以遇到listview的item点击无响应的时候，原因多半是由于在你自己定义的Item中存在诸如ImageButton，Button，CheckBox等子控件(也可以说是Button或者Checkable的子类控件)，
此时这些子控件会将焦点获取到，所以常常当点击item时变化的是子控件，item本身的点击没有响应。
这时在Item布局的根布局加上android:descendantFocusability=”blocksDescendants”的属性就好了。