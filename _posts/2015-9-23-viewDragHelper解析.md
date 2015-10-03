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

* onViewPositionChanged

当captureview的位置发生改变时回调

* onViewCaptured

当captureview被捕获时回调

* onEdgeTouched

当触摸到边界时回调。

* onEdgeLock

true的时候会锁住当前的边界，false则unLock。

* getOrderedChildIndex

改变同一个坐标（x,y）去寻找captureView位置的方法。（具体在：findTopChildUnder方法中）

总结下，方法的大致的回调顺序：

    {% highlight c %}
shouldInterceptTouchEvent：

DOWN:
    getOrderedChildIndex(findTopChildUnder)
    ->onEdgeTouched

MOVE:
    getOrderedChildIndex(findTopChildUnder)
    ->getViewHorizontalDragRange &
      getViewVerticalDragRange(checkTouchSlop)(MOVE中可能不止一次)
    ->clampViewPositionHorizontal&
      clampViewPositionVertical
    ->onEdgeDragStarted
    ->tryCaptureView
    ->onViewCaptured
    ->onViewDragStateChanged

processTouchEvent:

DOWN:
    getOrderedChildIndex(findTopChildUnder)
    ->tryCaptureView
    ->onViewCaptured
    ->onViewDragStateChanged
    ->onEdgeTouched
MOVE:
    ->STATE==DRAGGING:dragTo
    ->STATE!=DRAGGING:
        onEdgeDragStarted
        ->getOrderedChildIndex(findTopChildUnder)
        ->getViewHorizontalDragRange&
          getViewVerticalDragRange(checkTouchSlop)
        ->tryCaptureView
        ->onViewCaptured
        ->onViewDragStateChanged

      {% endhighlight %}

ok，上述是正常情况下大致的流程，当然整个过程可能会存在很多判断不成立的情况。

从上面也可以解释，我们在之前TextView(clickable=false)的情况下，没有编写getViewHorizontalDragRange方法时，
是可以移动的。因为直接进入processTouchEvent的DOWN，然后就onViewCaptured、onViewDragStateChanged（进入DRAGGING状态），
接下来MOVE就直接dragTo了。

而当子View消耗事件的时候，就需要走shouldInterceptTouchEvent，MOVE的时候经过一系列的判断（getViewHorizontalDragRange，
clampViewPositionVertical等），才能够去tryCaptureView。

## 源码分析

### ViewDragHelper实例的创建

ViewDragHelper重载了两个create()静态方法，先看两个参数的create()方法：

    {% highlight java %}
/**
 * Factory method to create a new ViewDragHelper.
 *
 * @param forParent Parent view to monitor
 * @param cb Callback to provide information and receive events
 * @return a new ViewDragHelper instance
 */
public static ViewDragHelper create(ViewGroup forParent, Callback cb) {
	return new ViewDragHelper(forParent.getContext(), forParent, cb);
}

    {% endhighlight %}

create()的两个参数很好理解，第一个是我们自定义的ViewGroup，第二个是控制子View拖拽需要的回调对象。create()直接调用了ViewDragHelper构造方法，
我们再来看看这个构造方法。

    {% highlight java %}
private ViewDragHelper(Context context, ViewGroup forParent, Callback cb) {
        if (forParent == null) {
            throw new IllegalArgumentException("Parent view may not be null");
        }
        if (cb == null) {
            throw new IllegalArgumentException("Callback may not be null");
        }

        mParentView = forParent;
        mCallback = cb;

        final ViewConfiguration vc = ViewConfiguration.get(context);
        final float density = context.getResources().getDisplayMetrics().density;
        mEdgeSize = (int) (EDGE_SIZE * density + 0.5f);

        mTouchSlop = vc.getScaledTouchSlop();
        mMaxVelocity = vc.getScaledMaximumFlingVelocity();
        mMinVelocity = vc.getScaledMinimumFlingVelocity();
        mScroller = ScrollerCompat.create(context, sInterpolator);
    }

    {% endhighlight %}

这个构造函数是私有的，也是仅有的构造函数，所以外部只能通过create()工厂方法来创建ViewDragHelper实例了。
这里要求了我们传递的自定义ViewGroup和回调对象不能为空，否则会直接抛出异常中断程序。在这里也初始化了一些触摸滑动需要的参考值和辅助类。

* mParentView和mCallback分别保存传递过来的对应参数
* ViewConfiguration类里定义了View相关的一系列时间、大小、距离等常量
* mEdgeSize表示边缘触摸的范围。例如mEdgeSize为20dp并且用户注册监听了左侧边缘触摸时，触摸点的x坐标小于mParentView.getLeft() +
mEdgeSize时（即触摸点在容器左边界往右20dp内）就算做是左侧的边缘触摸，详见ViewDragHelper的getEdgesTouched()方法。
* mTouchSlop是一个很小的距离值，只有在前后两次触摸点的距离超过mTouchSlop
的值时，我们才把这两次触摸算作是“滑动”，我们只在此时进行滑动处理，否则任何微小的距离的变化我们都要处理的话会显得太频繁，如果处理过程又比较复杂耗时就会使界面产生卡顿。
* mMaxVelocity、mMinVelocity是fling时的最大、最小速率，单位是像素每秒。
* mScroller是View滚动的辅助类

再看三个参数的create()方法：

    {% highlight java %}
    /**
     * Factory method to create a new ViewDragHelper.
     *
     * @param forParent Parent view to monitor
     * @param sensitivity Multiplier for how sensitive the helper should be about detecting
     *                    the start of a drag. Larger values are more sensitive. 1.0f is normal.
     * @param cb Callback to provide information and receive events
     * @return a new ViewDragHelper instance
     */
    public static ViewDragHelper create(ViewGroup forParent, float sensitivity, Callback cb) {
    	final ViewDragHelper helper = create(forParent, cb);
    	helper.mTouchSlop = (int) (helper.mTouchSlop * (1 / sensitivity));
    	return helper;
    }
    {% endhighlight %}

第二个参数sensitivity是用来调节mTouchSlop的值。sensitivity越大，mTouchSlop越小，对滑动的检测就越敏感。
例如sensitivity为1时，前后触摸点距离超过20dp才进行滑动处理，现在sensitivity为2的话，前后触摸点距离超过10dp就进行处理了。

### 对Touch事件的处理

当mParentView（自定义ViewGroup）被触摸时，首先会调用mParentView的onInterceptTouchEvent(MotionEvent ev)，
接着就调用shouldInterceptTouchEvent(MotionEvent ev) ，所以先来看看这个方法的ACTION_DOWN部分：

    {% highlight java %}
/**
 * Check if this event as provided to the parent view's onInterceptTouchEvent should
 * cause the parent to intercept the touch event stream.
 *
 * @param ev MotionEvent provided to onInterceptTouchEvent
 * @return true if the parent view should return true from onInterceptTouchEvent
 */
public boolean shouldInterceptTouchEvent(MotionEvent ev) {
    final int action = MotionEventCompat.getActionMasked(ev);
    final int actionIndex = MotionEventCompat.getActionIndex(ev);

    if (action == MotionEvent.ACTION_DOWN) {
        // Reset things for a new event stream, just in case we didn't get
        // the whole previous stream.
        cancel();
    }

    if (mVelocityTracker == null) {
        mVelocityTracker = VelocityTracker.obtain();
    }
    mVelocityTracker.addMovement(ev);

    switch (action) {
        case MotionEvent.ACTION_DOWN: {
            final float x = ev.getX();
            final float y = ev.getY();
            final int pointerId = MotionEventCompat.getPointerId(ev, 0);
            saveInitialMotion(x, y, pointerId);

            final View toCapture = findTopChildUnder((int) x, (int) y);

            // Catch a settling view if possible.
            if (toCapture == mCapturedView && mDragState == STATE_SETTLING) {
                tryCaptureViewForDrag(toCapture, pointerId);
            }

            final int edgesTouched = mInitialEdgesTouched[pointerId];
            if ((edgesTouched & mTrackingEdges) != 0) {
                mCallback.onEdgeTouched(edgesTouched & mTrackingEdges, pointerId);
            }
            break;
        }

		// 其他case暂且省略
    }

    return mDragState == STATE_DRAGGING;
}
    {% endhighlight %}

看9~21行，首先是关于多点触控.mVelocityTracker记录下触摸的各个点信息，稍后可以用来计算本次滑动的速率，每次发生ACTION_DOWN事件都会调用cancel()，
而在cancel()方法里mVelocityTracker又被清空了，所以mVelocityTracker记录下的是本次ACTION_DOWN事件直至ACTION_UP事件发生后
（下次ACTION_DOWN事件发生前）的所有触摸点的信息。

再来看24~42行case MotionEvent.ACTION_DOWN部分，先是调用saveInitialMotion(x, y, pointerId)保存手势的初始信息，即ACTION_DOWN发生时的触摸点坐标（x、y）、
触摸手指编号（pointerId），如果触摸到了mParentView的边缘还会记录触摸的是哪个边缘。接着调用findTopChildUnder((int) x, (int) y);
来获取当前触摸点下最顶层的子View，看findTopChildUnder的源码：

    {% highlight java %}
/**
 * Find the topmost child under the given point within the parent view's coordinate system.
 * The child order is determined using {@link Callback#getOrderedChildIndex(int)}.
 *
 * @param x X position to test in the parent's coordinate system
 * @param y Y position to test in the parent's coordinate system
 * @return The topmost child view under (x, y) or null if none found.
 */
public View findTopChildUnder(int x, int y) {
    final int childCount = mParentView.getChildCount();
    for (int i = childCount - 1; i >= 0; i--) {
        final View child = mParentView.getChildAt(mCallback.getOrderedChildIndex(i));
        if (x >= child.getLeft() && x < child.getRight() &&
                y >= child.getTop() && y < child.getBottom()) {
            return child;
        }
    }
    return null;
}

    {% endhighlight %}

代码很简单，注释里也说明的很清楚了。如果在同一个位置有两个子View重叠，想要让下层的子View被选中，
那么就要实现Callback里的getOrderedChildIndex(int index)方法来改变查找子View的顺序；例如topView（上层View）的index是4，
bottomView（下层View）的index是3，按照正常的遍历查找方式（getOrderedChildIndex()默认直接返回index），会选择到topView，
要想让bottomView被选中就得这么写：

    {% highlight java %}
public int getOrderedChildIndex(int index) {
	int indexTop = mParentView.indexOfChild(topView);
	int indexBottom = mParentView.indexOfChild(bottomView);
	if (index == indexTop) {
		return indexBottom;
	}
	return index;
}
    {% endhighlight %}

32~35行，这里还看到了一个mDragState成员变量，它共有三种取值：

1.STATE_IDLE：所有的View处于静止空闲状态
2.STATE_DRAGGING：某个View正在被用户拖动（用户正在与设备交互）
3.STATE_SETTLING：某个View正在安置状态中（用户并没有交互操作），就是自动滚动的过程中

mCapturedView默认为null，所以一开始不会执行这里的代码，mDragState处于STATE_SETTLING状态时才会执行tryCaptureViewForDrag()，
执行的情况到后面再分析.

37~40行调用了Callback.onEdgeTouched向外部通知mParentView的某些边缘被触摸到了，mInitialEdgesTouched是在刚才调用过的saveInitialMotion方法里进行赋值的。

