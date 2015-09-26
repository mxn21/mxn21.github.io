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

ViewDragHelper一般用在一个自定义ViewGroup的内部，比如下面自定义了一个继承于LinearLayout的MyLayout，
MyLayout内部有一个子viewmDragView作为成员变量,创建一个带有回调接口的ViewDragHelper.
其中1.0f是敏感度参数参数越大越敏感。第一个参数为this，表示该类生成的对象，他是ViewDragHelper的拖动处理对象，必须为ViewGroup。
第三个参数就是Callback，在用户的触摸过程中会回调相关方法.

   {% highlight java  %}

public class  MyLayout extends LinearLayout
{
    private ViewDragHelper mDragger;

    public MyLayout(Context context, AttributeSet attrs)
    {
        super(context, attrs);
        mDragger = ViewDragHelper.create(this, 1.0f, new ViewDragHelper.Callback()
        {
            @Override
            public boolean tryCaptureView(View child, int pointerId)
            {
                return true;
            }

            @Override
            public int clampViewPositionHorizontal(View child, int left, int dx)
            {
                return left;
            }

            @Override
            public int clampViewPositionVertical(View child, int top, int dy)
            {
                return top;
            }
        });
    }

   @Override
    public boolean onInterceptTouchEvent(MotionEvent event)
    {

        return mDragger.shouldInterceptTouchEvent(event);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event)
    {
        mDragger.processTouchEvent(event);
        return true;
    }
}

   {% endhighlight %}

要让ViewDragHelper能够处理拖动需要将触摸事件传递给ViewDragHelper，这点和gesturedetector是一样的.
onInterceptTouchEvent中通过使用mDragger.shouldInterceptTouchEvent(event)来决定我们是否应该拦截当前的事件。
onTouchEvent中通过mDragger.processTouchEvent(event)处理事件。

ViewDragHelper中拦截和处理事件时，需要会回调CallBack中的很多方法来决定一些事，比如：哪些子View可以移动、对个移动的View的边界的控制等等。

上面复写的3个方法：

* tryCaptureView如何返回ture则表示可以捕获该view，你可以根据传入的第一个view参数决定哪些可以捕获
* clampViewPositionHorizontal,clampViewPositionVertical可以在该方法中对child移动的边界进行控制，
left , top 分别为即将移动到的位置，比如横向的情况下，我希望只在ViewGroup的内部移动，即：最小>=paddingleft，
最大<=ViewGroup.getWidth()-paddingright-child.getWidth,就可以按照如下代码编写：


   {% highlight java  %}

   @Override
   public int clampViewPositionHorizontal(View child, int left, int dx)
    {
     final int leftBound = getPaddingLeft();
     final int rightBound = getWidth() - mDragView.getWidth() - leftBound;
     final int newLeft = Math.min(Math.max(left, leftBound), rightBound);
     return newLeft;
               }
   {% endhighlight %}

这样我们就完成了一个简单的自定义ViewGroup，可以自由的拖动子View。

2.布局文件

    {% highlight xml %}

  <com.mxn.view.MyLayout xmlns:android="http://schemas.android.com/apk/res/android"
                                xmlns:tools="http://schemas.android.com/tools"
                                android:layout_width="match_parent"
                                android:orientation="vertical"
                                android:layout_height="match_parent"
      >

      <TextView
          android:layout_margin="10dp"
          android:gravity="center"
          android:layout_gravity="center"
          android:background="#44ff0000"
          android:text="I can be dragged !"
          android:layout_width="100dp"
          android:layout_height="100dp"/>

      <TextView
          android:layout_margin="10dp"
          android:layout_gravity="center"
          android:gravity="center"
          android:background="#44ff0000"
          android:text="I can be dragged !"
          android:layout_width="100dp"
          android:layout_height="100dp"/>

      <TextView
          android:layout_margin="10dp"
          android:layout_gravity="center"
          android:gravity="center"
          android:background="#44ff0000"
          android:text="I can be dragged !"
          android:layout_width="100dp"
          android:layout_height="100dp"/>

  </com.mxn.view.MyLayout>

    {% endhighlight %}

我们的自定义ViewGroup中有三个TextView。能够去让子View去跟随我们手指移动.


注意在DragHelperCallback中实现clampViewPositionHorizontal方法， 并且返回一个适当的数值就能实现横向拖动效果，
clampViewPositionHorizontal的第二个参数是指当前拖动子view应该到达的x坐标。所以按照常理这个方法原封返回第二个参数就可以了，
但为了让被拖动的view遇到边界之后就不在拖动，对返回的值做了更多的考虑。

    {% highlight java %}
@Override
public int clampViewPositionHorizontal(View child, int left, int dx) {
  Log.d("DragLayout", "clampViewPositionHorizontal " + left + "," + dx);
  final int leftBound = getPaddingLeft();
  final int rightBound = getWidth() - mDragView.getWidth();
  final int newLeft = Math.min(Math.max(left, leftBound), rightBound);
  return newLeft;
}
    {% endhighlight %}

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img111.gif)


同上，处理纵向的拖动：

在DragHelperCallback中实现clampViewPositionVertical方法，实现过程同clampViewPositionHorizontal

    {% highlight java %}
@Override
public int clampViewPositionVertical(View child, int top, int dy) {
  final int topBound = getPaddingTop();
  final int bottomBound = getHeight() - mDragView.getHeight();
  final int newTop = Math.min(Math.max(top, topBound), bottomBound);
  return newTop;
}
    {% endhighlight %}

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img112.gif)

clampViewPositionHorizontal 和 clampViewPositionVertical必须要重写，因为默认它返回的是0。事实上我们在这两个方法中所能做的事情很有限。
这两个方法的作用就是给了我们重新定义目的坐标的机会。

通过DragHelperCallback的tryCaptureView方法的返回值可以决定一个parentview中哪个子view可以拖动，现在假设有两个子views
(mDragView1和mDragView2)  ，如下实现tryCaptureView之后，则只有mDragView1是可以拖动的。

    {% highlight java %}
@Override
public boolean tryCaptureView(View child, int pointerId) {
  returnchild == mDragView1;
}
    {% endhighlight %}

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img113.gif)

滑动边缘:
分为滑动左边缘还是右边缘：EDGE_LEFT和EDGE_RIGHT，下面的代码设置了可以处理滑动左边缘：

    {% highlight java %}
mDragHelper.setEdgeTrackingEnabled(ViewDragHelper.EDGE_LEFT);
    {% endhighlight %}

假如如上设置，onEdgeTouched方法会在左边缘滑动的时候被调用，这种情况下一般都是没有和子view接触的情况。

    {% highlight java %}
@Override
public void onEdgeTouched(int edgeFlags, int pointerId) {
    super.onEdgeTouched(edgeFlags, pointerId);
    Toast.makeText(getContext(), "edgeTouched", Toast.LENGTH_SHORT).show();
}
      {% endhighlight %}

如果你想在边缘滑动的时候根据滑动距离移动一个子view，可以通过实现onEdgeDragStarted方法，并在onEdgeDragStarted方法中手动指定要移动的子View

    {% highlight java %}
@Override
public void onEdgeDragStarted(int edgeFlags, int pointerId) {
    mDragHelper.captureChildView(mDragView2, pointerId);
}
      {% endhighlight %}

我们在onEdgeDragStarted回调方法中，主动通过captureChildView对其进行捕获，该方法可以绕过tryCaptureView，所以我们的tryCaptureView
虽然并未返回true，但却不影响。注意如果需要使用边界检测需要添加上mDragger.setEdgeTrackingEnabled(ViewDragHelper.EDGE_LEFT);。

手指释放的时候回调：

    {% highlight java %}
        @Override
        public void onViewReleased(View releasedChild, float xvel, float yvel)
        {
                //mAutoBackView手指释放时可以自动回去
                if (releasedChild == mAutoBackView)
                {
                    mDragger.settleCapturedViewAt(mAutoBackOriginPos.x, mAutoBackOriginPos.y);
                    invalidate();
                }
       }

      {% endhighlight %}

重写了Callback中的onViewReleased，我们在onViewReleased中判断如果是mAutoBackView则调用settleCapturedViewAt回到初始的位置。
大家可以看到紧随其后的代码是invalidate();因为其内部使用的是mScroller.startScroll，所以别忘了需要invalidate()以及结合computeScroll方法一起。

需要加上：

    {% highlight java %}
    @Override
    public void computeScroll()
    {
        if(mDragger.continueSettling(true))
        {
            invalidate();
        }
    }
      {% endhighlight %}

到此，我们已经介绍了Callback中常用的回调方法了，当然还有一些方法没有介绍，接下来我们修改下我们的布局文件，
我们把我们的TextView全部加上clickable=true，意思就是子View可以消耗事件。再次运行，你会发现本来可以拖动的View不动了。

原因是什么呢？主要是因为，如果子View不消耗事件，那么整个手势（DOWN-MOVE*-UP）都是直接进入onTouchEvent，在onTouchEvent的DOWN的时候就确定了captureView。
如果消耗事件，那么就会先走onInterceptTouchEvent方法，判断是否可以捕获，而在判断的过程中会去判断另外两个回调的方法：
getViewHorizontalDragRange和getViewVerticalDragRange，只有这两个方法返回大于0的值才能正常的捕获。

所以，如果你用Button测试，或者给TextView添加了clickable = true ，都记得重写下面这两个方法：

    {% highlight java %}
@Override
public int getViewHorizontalDragRange(View child)
{
     return getMeasuredWidth()-child.getMeasuredWidth();
}

@Override
public int getViewVerticalDragRange(View child)
{
     return getMeasuredHeight()-child.getMeasuredHeight();
}
      {% endhighlight %}

方法的返回值应当是该childView横向或者纵向的移动的范围，当前如果只需要一个方向移动，可以只复写一个。

到此，我们列一下所有的Callback方法，看看还有哪些没用过的：

* onViewDragStateChanged

当ViewDragHelper状态发生变化时回调（IDLE,DRAGGING,SETTING[自动滚动时]）

onViewPositionChanged

当captureview的位置发生改变时回调


