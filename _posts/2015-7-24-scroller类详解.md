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

### Scroller类介绍

我们就分析下源码里去看看Scroller类的相关方法.
其源代码(部分)如下： 路径位于 \frameworks\base\core\java\android\widget\Scroller.java

    {% highlight java  %}

public class Scroller  {

    private int mStartX;    //起始坐标点 ,  X轴方向
    private int mStartY;    //起始坐标点 ,  Y轴方向
    private int mCurrX;     //当前坐标点  X轴， 即调用startScroll函数后,经过一定时间所达到的值
    private int mCurrY;     //当前坐标点  Y轴， 即调用startScroll函数后,经过一定时间所达到的值

    private float mDeltaX;  //应该继续滑动的距离， X轴方向
    private float mDeltaY;  //应该继续滑动的距离， Y轴方向
    private boolean mFinished;  //是否已经完成本次滑动操作， 如果完成则为 true

    //构造函数
    public Scroller(Context context) {
        this(context, null);
    }
    public final boolean isFinished() {
        return mFinished;
    }
    //强制结束本次滑屏操作
    public final void forceFinished(boolean finished) {
        mFinished = finished;
    }
    public final int getCurrX() {
        return mCurrX;
    }
     /* Call this when you want to know the new location.  If it returns true,
     * the animation is not yet finished.  loc will be altered to provide the
     * new location. */
    //根据当前已经消逝的时间计算当前的坐标点，保存在mCurrX和mCurrY值中
    public boolean computeScrollOffset() {
        if (mFinished) {  //已经完成了本次动画控制，直接返回为false
            return false;
        }
        int timePassed = (int)(AnimationUtils.currentAnimationTimeMillis() - mStartTime);
        if (timePassed < mDuration) {
            switch (mMode) {
            case SCROLL_MODE:
                float x = (float)timePassed * mDurationReciprocal;
                ...
                mCurrX = mStartX + Math.round(x * mDeltaX);
                mCurrY = mStartY + Math.round(x * mDeltaY);
                break;
            ...
        }
        else {
            mCurrX = mFinalX;
            mCurrY = mFinalY;
            mFinished = true;
        }
        return true;
    }
    //开始一个动画控制，由(startX , startY)在duration时间内前进(dx,dy)个单位，即到达坐标为(startX+dx , startY+dy)出
    public void startScroll(int startX, int startY, int dx, int dy, int duration) {
        mFinished = false;
        mDuration = duration;
        mStartTime = AnimationUtils.currentAnimationTimeMillis();
        mStartX = startX;       mStartY = startY;
        mFinalX = startX + dx;  mFinalY = startY + dy;
        mDeltaX = dx;            mDeltaY = dy;
        ...
    }
}
    {% endhighlight %}


<!-- more -->

其中比较重要的两个方法为：

public boolean computeScrollOffset()

函数功能说明：根据当前已经消逝的时间计算当前的坐标点，保存在mCurrX和mCurrY值中。

public void startScroll(int startX, int startY, int dx, int dy, int duration)

函数功能说明：开始一个动画控制，由(startX , startY)在duration时间内前进(dx,dy)个单位，到达坐标为(startX+dx , startY+dy)处。


### computeScroll(）方法介绍

为了易于控制滑屏控制，Android框架提供了 computeScroll()方法去控制这个流程。在绘制View时，会在draw()过程调用该
方法。因此， 再配合使用Scroller实例，我们就可以获得当前应该的偏移坐标，手动使View/ViewGroup偏移至该处。
computeScroll()方法原型如下，该方法位于ViewGroup.java类中

    {% highlight java  %}
/**
     * Called by a parent to request that a child update its values for mScrollX
     * and mScrollY if necessary. This will typically be done if the child is
     * animating a scroll using a {@link android.widget.Scroller Scroller}
     * object.
     */由父视图调用用来请求子视图根据偏移值 mScrollX,mScrollY重新绘制
    public void computeScroll() { //空方法 ，自定义ViewGroup必须实现方法体

    }
    {% endhighlight %}

为了实现偏移控制，一般自定义View/ViewGroup都需要重载该方法 。
其调用过程位于View绘制流程draw()过程中，如下：

    {% highlight java  %}
@Override
protected void dispatchDraw(Canvas canvas){
    ...

    for (int i = 0; i < count; i++) {
        final View child = children[getChildDrawingOrder(count, i)];
        if ((child.mViewFlags & VISIBILITY_MASK) == VISIBLE || child.getAnimation() != null) {
            more |= drawChild(canvas, child, drawingTime);
        }
    }
}
protected boolean drawChild(Canvas canvas, View child, long drawingTime) {
    ...
    child.computeScroll();
    ...
}

    {% endhighlight %}


### ViewConfiguration类

功能： 获得一些关于timeouts(时间)、sizes(大小)、distances(距离)的标准常量值 。

常用方法：

public int getScaledEdgeSlop

说明：获得一个触摸移动的最小像素值。也就是说，只有超过了这个值，才代表我们该滑屏处理了

public static int getLongPressTimeout()

说明：获得一个执行长按事件监听(onLongClickListener)的值。也就是说，对某个View按下触摸时，只有超过了
这个时间值在，才表示我们该对该View回调长按事件了；否则，小于这个时间点松开手指，只执行onClick监听。


### VelocityTracker类

功能：  根据触摸位置计算每像素的移动速率。

常用方法有：

public void addMovement (MotionEvent ev)

功能：添加触摸对象MotionEvent ， 用于计算触摸速率。

public void computeCurrentVelocity (int units)

功能：以每像素units单位考核移动速率。

public float getXVelocity ()

功能：获得X轴方向的移动速率。


### getScaledTouchSlop()

返回一个距离，表示滑动的时候，手的移动要大于这个距离才开始移动控件。

调用方法：

ViewConfiguration.get(getContext()).getScaledTouchSlop();


