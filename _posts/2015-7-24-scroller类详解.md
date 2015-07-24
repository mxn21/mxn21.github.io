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




