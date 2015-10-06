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

ACTION_DOWN部分处理完了，跳过switch语句块，剩下的代码就只有return mDragState == STATE_DRAGGING;。在ACTION_DOWN部分没有对mDragState进行赋值，其默认值为STATE_IDLE，所以此处返回false。

那么返回false后接下来应该是会调用哪个方法呢,接下来会在mParentView的所有子View中寻找响应这个Touch事件的View（会调用每个子View
的dispatchTouchEvent()方法，dispatchTouchEvent里一般又会调用onTouchEvent()）.

1.如果没有子View消费这次事件（子View的dispatchTouchEvent()返回都是false），会调用mParentView的super.dispatchTouchEvent
(ev)，即View中的dispatchTouchEvent(ev)，然后调用mParentView的onTouchEvent()方法，
再调用ViewDragHelper的processTouchEvent(MotionEvent ev)方法。此时（ACTION_DOWN事件发生时）mParentView的onTouchEvent()要返回true，
onTouchEvent()才能继续接受到接下来的ACTION_MOVE、ACTION_UP等事件，否则无法完成拖动（除了ACTION_DOWN外的其他事件发生时返回true或false都
不会影响接下来的事件接受），因为拖动的相关代码是写在processTouchEvent()里的ACTION_MOVE部分的。
要注意的是返回true后mParentView的onInterceptTouchEvent()就不会收到后续的ACTION_MOVE、ACTION_UP等事件了。

2.如果有子View消费了本次ACTION_DOWN事件，mParentView的onTouchEvent()就收不到ACTION_DOWN事件了，
也就是ViewDragHelper的processTouchEvent(MotionEvent ev)收不到ACTION_DOWN事件了。
不过只要该View没有调用过requestDisallowInterceptTouchEvent(true)，mParentView的onInterceptTouchEvent()的ACTION_MOVE部分还是会执行的，
如果在此时返回了true拦截了ACTION_MOVE事件，processTouchEvent()里的ACTION_MOVE部分也就会正常执行，拖动也就没问题了。
onInterceptTouchEvent()的ACTION_MOVE部分具体做了怎样的处理，稍后再来解析。

接下来对这两种情况逐一解析。

假设没有子View消费这次事件，根据刚才的分析最终就会调用processTouchEvent(MotionEvent ev)的ACTION_DOWN部分：

    {% highlight java %}
    /**
     * Process a touch event received by the parent view. This method will dispatch callback events
     * as needed before returning. The parent view's onTouchEvent implementation should call this.
     *
     * @param ev The touch event received by the parent view
     */
    public void processTouchEvent(MotionEvent ev) {
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
                final View toCapture = findTopChildUnder((int) x, (int) y);

                saveInitialMotion(x, y, pointerId);

                // Since the parent is already directly processing this touch event,
                // there is no reason to delay for a slop before dragging.
                // Start immediately if possible.
                tryCaptureViewForDrag(toCapture, pointerId);

                final int edgesTouched = mInitialEdgesTouched[pointerId];
                if ((edgesTouched & mTrackingEdges) != 0) {
                    mCallback.onEdgeTouched(edgesTouched & mTrackingEdges, pointerId);
                }
                break;
            }
    		// 其他case暂且省略
        }
    }
    {% endhighlight %}

这段代码跟shouldInterceptTouchEvent()里ACTION_DOWN那部分基本一致，唯一区别就是这里没有约束条件直接调用了tryCaptureViewForDrag()方法，现在来看看这个方法：

    {% highlight java %}
/**
 * Attempt to capture the view with the given pointer ID. The callback will be involved.
 * This will put us into the "dragging" state. If we've already captured this view with
 * this pointer this method will immediately return true without consulting the callback.
 *
 * @param toCapture View to capture
 * @param pointerId Pointer to capture with
 * @return true if capture was successful
 */
boolean tryCaptureViewForDrag(View toCapture, int pointerId) {
    if (toCapture == mCapturedView && mActivePointerId == pointerId) {
        // Already done!
        return true;
    }
    if (toCapture != null && mCallback.tryCaptureView(toCapture, pointerId)) {
        mActivePointerId = pointerId;
        captureChildView(toCapture, pointerId);
        return true;
    }
    return false;
}
    {% endhighlight %}

这里调用了Callback的tryCaptureView(View child, int pointerId)方法，把当前触摸到的View和触摸手指编号传递了过去，在tryCaptureView()中决定是否需要拖动当前触摸到的View，如果要拖动当前触摸到的View就在tryCaptureView()中返回true，让ViewDragHelper把当前触摸的View捕获下来，
接着就调用了captureChildView(toCapture, pointerId)方法：

    {% highlight java %}
/**
 * Capture a specific child view for dragging within the parent. The callback will be notified
 * but {@link Callback#tryCaptureView(android.view.View, int)} will not be asked permission to
 * capture this view.
 *
 * @param childView Child view to capture
 * @param activePointerId ID of the pointer that is dragging the captured child view
 */
public void captureChildView(View childView, int activePointerId) {
    if (childView.getParent() != mParentView) {
        throw new IllegalArgumentException("captureChildView: parameter must be a descendant " +
                "of the ViewDragHelper's tracked parent view (" + mParentView + ")");
    }

    mCapturedView = childView;
    mActivePointerId = activePointerId;
    mCallback.onViewCaptured(childView, activePointerId);
    setDragState(STATE_DRAGGING);
}
    {% endhighlight %}

代码很简单，在captureChildView(toCapture, pointerId)中将要拖动的View和触摸的手指编号记录下来，并调用Callback的onViewCaptured(childView, activePointerId)通知外部有子View被捕获到了，
再调用setDragState()设置当前的状态为STATE_DRAGGING，看setDragState()源码：

    {% highlight java %}
void setDragState(int state) {
    if (mDragState != state) {
        mDragState = state;
        mCallback.onViewDragStateChanged(state);
        if (mDragState == STATE_IDLE) {
            mCapturedView = null;
        }
    }
}
    {% endhighlight %}

状态改变后会调用Callback的onViewDragStateChanged()通知状态的变化。

假设ACTION_DOWN发生后在mParentView的onTouchEvent()返回了true，接下来就会执行ACTION_MOVE部分：

    {% highlight java %}
public void processTouchEvent(MotionEvent ev) {

    switch (action) {
        // 省略其他case...

        case MotionEvent.ACTION_MOVE: {
            if (mDragState == STATE_DRAGGING) {
                final int index = MotionEventCompat.findPointerIndex(ev, mActivePointerId);
                final float x = MotionEventCompat.getX(ev, index);
                final float y = MotionEventCompat.getY(ev, index);
                final int idx = (int) (x - mLastMotionX[mActivePointerId]);
                final int idy = (int) (y - mLastMotionY[mActivePointerId]);

                dragTo(mCapturedView.getLeft() + idx, mCapturedView.getTop() + idy, idx, idy);

                saveLastMotion(ev);
            } else {
                // Check to see if any pointer is now over a draggable view.
                final int pointerCount = MotionEventCompat.getPointerCount(ev);
                for (int i = 0; i < pointerCount; i++) {
                    final int pointerId = MotionEventCompat.getPointerId(ev, i);
                    final float x = MotionEventCompat.getX(ev, i);
                    final float y = MotionEventCompat.getY(ev, i);
                    final float dx = x - mInitialMotionX[pointerId];
                    final float dy = y - mInitialMotionY[pointerId];

                    reportNewEdgeDrags(dx, dy, pointerId);
                    if (mDragState == STATE_DRAGGING) {
                        // Callback might have started an edge drag.
                        break;
                    }

                    final View toCapture = findTopChildUnder((int) x, (int) y);
                    if (checkTouchSlop(toCapture, dx, dy) &&
                            tryCaptureViewForDrag(toCapture, pointerId)) {
                        break;
                    }
                }
                saveLastMotion(ev);
            }
            break;
        }

		// 省略其他case...
    }
}
    {% endhighlight %}

要注意的是，如果一直没松手，这部分代码会一直调用。这里先判断mDragState是否为STATE_DRAGGING，而唯一调用setDragState(STATE_DRAGGING)的地方就是tryCaptureViewForDrag()了，
刚才在ACTION_DOWN里调用过tryCaptureViewForDrag()，现在又要分两种情况。

如果刚才在ACTION_DOWN里捕获到要拖动的View，那么就执行if部分的代码，这个稍后解析，先考虑没有捕获到的情况。没有捕获到的话，mDragState依然是STATE_IDLE，然后会执行else部分的代码。这里主要就是检查有没有哪个手指触摸到了要拖动的View上，触摸上了就尝试捕获它，然后让mDragState变为STATE_DRAGGING，之后就会执行if部分的代码了。这里还有两个方法涉及到了Callback里的方法，需要来解析一下，
分别是reportNewEdgeDrags()和checkTouchSlop()，先看reportNewEdgeDrags()：

    {% highlight java %}
private void reportNewEdgeDrags(float dx, float dy, int pointerId) {
    int dragsStarted = 0;
    if (checkNewEdgeDrag(dx, dy, pointerId, EDGE_LEFT)) {
        dragsStarted |= EDGE_LEFT;
    }
    if (checkNewEdgeDrag(dy, dx, pointerId, EDGE_TOP)) {
        dragsStarted |= EDGE_TOP;
    }
    if (checkNewEdgeDrag(dx, dy, pointerId, EDGE_RIGHT)) {
        dragsStarted |= EDGE_RIGHT;
    }
    if (checkNewEdgeDrag(dy, dx, pointerId, EDGE_BOTTOM)) {
        dragsStarted |= EDGE_BOTTOM;
    }

    if (dragsStarted != 0) {
        mEdgeDragsInProgress[pointerId] |= dragsStarted;
        mCallback.onEdgeDragStarted(dragsStarted, pointerId);
    }
}
    {% endhighlight %}

这里对四个边缘都做了一次检查，检查是否在某些边缘产生拖动了，如果有拖动，就将有拖动的边缘记录在mEdgeDragsInProgress中，再调用Callback的onEdgeDragStarted(int edgeFlags, int pointerId)通知某个边缘开始产生拖动了。虽然reportNewEdgeDrags()会被调用很多次（因为processTouchEvent()的ACTION_MOVE部分会执行很多次），
但mCallback.onEdgeDragStarted(dragsStarted, pointerId)只会调用一次，具体的要看checkNewEdgeDrag()这个方法：

    {% highlight java %}
private boolean checkNewEdgeDrag(float delta, float odelta, int pointerId, int edge) {
    final float absDelta = Math.abs(delta);
    final float absODelta = Math.abs(odelta);

    if ((mInitialEdgesTouched[pointerId] & edge) != edge  || (mTrackingEdges & edge) == 0 ||
            (mEdgeDragsLocked[pointerId] & edge) == edge ||
            (mEdgeDragsInProgress[pointerId] & edge) == edge ||
            (absDelta <= mTouchSlop && absODelta <= mTouchSlop)) {
        return false;
    }
    if (absDelta < absODelta * 0.5f && mCallback.onEdgeLock(edge)) {
        mEdgeDragsLocked[pointerId] |= edge;
        return false;
    }
    return (mEdgeDragsInProgress[pointerId] & edge) == 0 && absDelta > mTouchSlop;
}
    {% endhighlight %}

* checkNewEdgeDrag()返回true表示在指定的edge（边缘）开始产生拖动了。
* 方法的两个参数delta和odelta需要解释一下，odelta里的o应该代表opposite，这是什么意思呢，以reportNewEdgeDrags()里调用checkNewEdgeDrag(dx, dy, pointerId, EDGE_LEFT)为例，我们要监测左边缘的触摸情况，所以主要监测的是x轴方向上的变化，这里delta为dx，odelta为dy，也就是说delta是指我们主要监测的方向上的变化，odelta是另外一个方向上的变化，后面要判断假另外一个方向上的变化是否要远大于主要方向上的变化，所以需要另外一个方向上的距离变化的值。
* mInitialEdgesTouched是在ACTION_DOWN部分的saveInitialMotion()里生成的，ACTION_DOWN发生时触摸到的边缘会被记录在mInitialEdgesTouched中。如果ACTION_DOWN发生时没有触摸到边缘，或者触摸到的边缘不是指定的edge，就直接返回false了。
* mTrackingEdges是由setEdgeTrackingEnabled(int edgeFlags)设置的，当我们想要追踪监听边缘触摸时才需要调用setEdgeTrackingEnabled(int edgeFlags)，如果我们没有调用过它，这里就直接返回false了。
* mEdgeDragsLocked它在这个方法里被引用了多次，它在整个ViewDragHelper里唯一被赋值的地方就是这里的第12行，所以默认值是0，第6行mEdgeDragsLocked[pointerId] & edge) == edge执行的结果是false。我们再跳到11到14行看看，absDelta < absODelta * 0.5f的意思是检查在次要方向上移动的距离是否远超过主要方向上移动的距离，如果是再调用Callback的onEdgeLock(edge)检查是否需要锁定某个边缘，如果锁定了某个边缘，那个边缘就算触摸到了也不会被记录在mEdgeDragsInProgress里了，也不会收到Callback的onEdgeDragStarted()通知了。并且将锁定的边缘记录在mEdgeDragsLocked变量里，再次调用本方法时就会在第6行进行判断了，第6行里如果检测到给定的edge被锁定，就直接返回false了。
* 回到第7行的(mEdgeDragsInProgress[pointerId] & edge) == edge，mEdgeDragsInProgress是保存已发生过拖动事件的边缘的，如果给定的edge已经保存过了，那就没必要再检测其他东西了，直接返回false了。
* 第8行(absDelta <= mTouchSlop && absODelta <= mTouchSlop)很简单了，就是检查本次移动的距离是不是太小了，太小就不处理了。
* 最后一句返回的时候再次检查给定的edge有没有记录过，确保了每个边缘只会调用一次reportNewEdgeDrags的mCallback.onEdgeDragStarted(dragsStarted, pointerId)

再来看checkTouchSlop()方法：

    {% highlight java %}
/**
 * Check if we've crossed a reasonable touch slop for the given child view.
 * If the child cannot be dragged along the horizontal or vertical axis, motion
 * along that axis will not count toward the slop check.
 *
 * @param child Child to check
 * @param dx Motion since initial position along X axis
 * @param dy Motion since initial position along Y axis
 * @return true if the touch slop has been crossed
 */
private boolean checkTouchSlop(View child, float dx, float dy) {
    if (child == null) {
        return false;
    }
    final boolean checkHorizontal = mCallback.getViewHorizontalDragRange(child) > 0;
    final boolean checkVertical = mCallback.getViewVerticalDragRange(child) > 0;

    if (checkHorizontal && checkVertical) {
        return dx * dx + dy * dy > mTouchSlop * mTouchSlop;
    } else if (checkHorizontal) {
        return Math.abs(dx) > mTouchSlop;
    } else if (checkVertical) {
        return Math.abs(dy) > mTouchSlop;
    }
    return false;
}
    {% endhighlight %}

这个方法主要就是检查手指移动的距离有没有超过触发处理移动事件的最短距离（mTouchSlop）了，注意dx和dy指的是当前触摸点到ACTION_DOWN触摸到的点的距离。这里先检查Callback的getViewHorizontalDragRange(child)和getViewVerticalDragRange(child)是否大于0，如果想让某个View在某个方向上滑动，就要在那个方向对应的方法里返回大于0的数。否则在processTouchEvent()的ACTION_MOVE部分就不会调用tryCaptureViewForDrag()来捕获当前触摸到的View了，拖动也就没办法进行了。

回到processTouchEvent()的ACTION_MOVE部分，假设现在我们的手指已经滑动到可以被捕获到的View上了，也都正常的实现了Callback中的相关方法，让tryCaptureViewForDrag()正常的捕获到触摸到的View了，下一次ACTION_MOVE时就执行if部分的代码了，也就是开始不停的调用dragTo()对mCaptureView进行真正拖动了，看dragTo()方法：

    {% highlight java %}
    private void dragTo(int left, int top, int dx, int dy) {
        int clampedX = left;
        int clampedY = top;
        final int oldLeft = mCapturedView.getLeft();
        final int oldTop = mCapturedView.getTop();
        if (dx != 0) {
            clampedX = mCallback.clampViewPositionHorizontal(mCapturedView, left, dx);
            mCapturedView.offsetLeftAndRight(clampedX - oldLeft);
        }
        if (dy != 0) {
            clampedY = mCallback.clampViewPositionVertical(mCapturedView, top, dy);
            mCapturedView.offsetTopAndBottom(clampedY - oldTop);
        }

        if (dx != 0 || dy != 0) {
            final int clampedDx = clampedX - oldLeft;
            final int clampedDy = clampedY - oldTop;
            mCallback.onViewPositionChanged(mCapturedView, clampedX, clampedY,
                    clampedDx, clampedDy);
        }
    }
    {% endhighlight %}

参数dx和dy是前后两次ACTION_MOVE移动的距离，left和top分别为mCapturedView.getLeft() + dx, mCapturedView.getTop() + dy，也就是期望的移动后的坐标，对View的getLeft()等方法不理解的请参阅Android View坐标getLeft, getRight, getTop, getBottom。

这里通过调用offsetLeftAndRight()和offsetTopAndBottom()来完成对mCapturedView移动，这两个是View中定义的方法，看它们的源码就知道内部是通过改变View的mLeft、mRight、mTop、mBottom，即改变View在父容器中的坐标位置，达到移动View的效果，所以如果调用mCapturedView的layout(int l, int t, int r, int b)方法也可以实现移动View的效果。

具体要移动到哪里，由Callback的clampViewPositionHorizontal()和clampViewPositionVertical()来决定的，如果不想在水平方向上移动，在clampViewPositionHorizontal(View child, int left, int dx)里直接返回child.getLeft()就可以了，这样clampedX - oldLeft的值为0，这里调用mCapturedView.offsetLeftAndRight(clampedX - oldLeft)就不会起作用了。垂直方向上同理。

最后会调用Callback的onViewPositionChanged(mCapturedView, clampedX, clampedY,clampedDx, clampedDy)通知捕获到的View位置改变了，并把最终的坐标（clampedX、clampedY）和最终的移动距离（clampedDx、 clampedDy）传递过去。

ACTION_MOVE部分就算告一段落了，接下来应该是用户松手触发ACTION_UP，或者是达到某个条件导致后续的ACTION_MOVE被mParentView的上层View给拦截了而收到ACTION_CANCEL，一起来看这两个部分：

    {% highlight java %}
public void processTouchEvent(MotionEvent ev) {
    // 省略

    switch (action) {
        // 省略其他case

        case MotionEvent.ACTION_UP: {
            if (mDragState == STATE_DRAGGING) {
                releaseViewForPointerUp();
            }
            cancel();
            break;
        }

        case MotionEvent.ACTION_CANCEL: {
            if (mDragState == STATE_DRAGGING) {
                dispatchViewReleased(0, 0);
            }
            cancel();
            break;
        }
    }
}
    {% endhighlight %}

这两个部分都是重置所有的状态记录，并通知View被放开了，再看下releaseViewForPointerUp()和dispatchViewReleased()的源码：

    {% highlight java %}
private void releaseViewForPointerUp() {
    mVelocityTracker.computeCurrentVelocity(1000, mMaxVelocity);
    final float xvel = clampMag(
            VelocityTrackerCompat.getXVelocity(mVelocityTracker, mActivePointerId),
            mMinVelocity, mMaxVelocity);
    final float yvel = clampMag(
            VelocityTrackerCompat.getYVelocity(mVelocityTracker, mActivePointerId),
            mMinVelocity, mMaxVelocity);
    dispatchViewReleased(xvel, yvel);
}

    {% endhighlight %}

releaseViewForPointerUp()里也调用了dispatchViewReleased()，只不过传递了速率给它，这个速率就是由processTouchEvent()的mVelocityTracker追踪算出来的。再看dispatchViewReleased()：


    {% highlight java %}
/**
 * Like all callback events this must happen on the UI thread, but release
 * involves some extra semantics. During a release (mReleaseInProgress)
 * is the only time it is valid to call {@link #settleCapturedViewAt(int, int)}
 * or {@link #flingCapturedView(int, int, int, int)}.
 */
private void dispatchViewReleased(float xvel, float yvel) {
    mReleaseInProgress = true;
    mCallback.onViewReleased(mCapturedView, xvel, yvel);
    mReleaseInProgress = false;

    if (mDragState == STATE_DRAGGING) {
        // onViewReleased didn't call a method that would have changed this. Go idle.
        setDragState(STATE_IDLE);
    }
}
    {% endhighlight %}

这里调用Callback的onViewReleased(mCapturedView, xvel, yvel)通知外部捕获到的View被释放了，而在onViewReleased()前后有个mReleaseInProgress值得注意，注释里说唯一可以调用ViewDragHelper的settleCapturedViewAt()和flingCapturedView()的地方就是在Callback的onViewReleased()里了。

首先这两个方法是干什么的呢。在现实生活中保龄球的打法是，先做扔的动作让球的速度达到最大，然后突然松手，由于惯性，保龄球就以最后松手前的速度为初速度抛出去了，直至自然停止，或者撞到边界停止，这种效果叫fling。
flingCapturedView(int minLeft, int minTop, int maxLeft, int maxTop)就是对捕获到的View做出这种fling的效果，用户在屏幕上滑动松手之前也会有一个滑动的速率。fling也引出来的一个问题，就是不知道View最终会滚动到哪个位置，最后位置是在启动fling时根据最后滑动的速度来计算的（flingCapturedView的四个参数int minLeft, int minTop, int maxLeft, int maxTop可以限定最终位置的范围），假如想要让View滚动到指定位置应该怎么办，答案就是使用settleCapturedViewAt(int finalLeft, int finalTop)。

为什么唯一可以调用settleCapturedViewAt()和flingCapturedView()的地方是Callback的onViewReleased()呢？看看它们的源码

    {% highlight java %}
/**
 * Settle the captured view at the given (left, top) position.
 * The appropriate velocity from prior motion will be taken into account.
 * If this method returns true, the caller should invoke {@link #continueSettling(boolean)}
 * on each subsequent frame to continue the motion until it returns false. If this method
 * returns false there is no further work to do to complete the movement.
 *
 * @param finalLeft Settled left edge position for the captured view
 * @param finalTop Settled top edge position for the captured view
 * @return true if animation should continue through {@link #continueSettling(boolean)} calls
 */
public boolean settleCapturedViewAt(int finalLeft, int finalTop) {
    if (!mReleaseInProgress) {
        throw new IllegalStateException("Cannot settleCapturedViewAt outside of a call to " +
                "Callback#onViewReleased");
    }

    return forceSettleCapturedViewAt(finalLeft, finalTop,
            (int) VelocityTrackerCompat.getXVelocity(mVelocityTracker, mActivePointerId),
            (int) VelocityTrackerCompat.getYVelocity(mVelocityTracker, mActivePointerId));
}

/**
 * Settle the captured view based on standard free-moving fling behavior.
 * The caller should invoke {@link #continueSettling(boolean)} on each subsequent frame
 * to continue the motion until it returns false.
 *
 * @param minLeft Minimum X position for the view's left edge
 * @param minTop Minimum Y position for the view's top edge
 * @param maxLeft Maximum X position for the view's left edge
 * @param maxTop Maximum Y position for the view's top edge
 */
public void flingCapturedView(int minLeft, int minTop, int maxLeft, int maxTop) {
    if (!mReleaseInProgress) {
        throw new IllegalStateException("Cannot flingCapturedView outside of a call to " +
                "Callback#onViewReleased");
    }

    mScroller.fling(mCapturedView.getLeft(), mCapturedView.getTop(),
            (int) VelocityTrackerCompat.getXVelocity(mVelocityTracker, mActivePointerId),
            (int) VelocityTrackerCompat.getYVelocity(mVelocityTracker, mActivePointerId),
            minLeft, maxLeft, minTop, maxTop);

    setDragState(STATE_SETTLING);
}

    {% endhighlight %}

这两个方法里一开始都会判断mReleaseInProgress为false，如果为false就会抛一个IllegalStateException异常，
而mReleaseInProgress唯一为true的时候就是在dispatchViewReleased()里调用onViewReleased()的时候。

