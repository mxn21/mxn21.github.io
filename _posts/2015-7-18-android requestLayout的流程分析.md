---
layout: post
author: mxn
titile: android requestLayout的流程分析
category: 技术博文
tag: android
---

当一个View调用requestLayout的时候，会给当前的View设置一个FORCE_LAYOUT标记。
由此向ViewParent请求布局。这样从这个View开始向上一直requestLayout。最终到达ViewRootImpl。ViewParent 就是当前的传输链。

ViewRootImpl发现请求了布局。那么就会调用measure方法。
measure方法确认当前View是否有FORCE_LAYOUT标记。
如果有，那么就会进行重新measure。并且设置标记LAYOUT_REQUIRED。

    {% highlight java  %}
    public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
        if ((mPrivateFlags & FORCE_LAYOUT) == FORCE_LAYOUT ||
                widthMeasureSpec != mOldWidthMeasureSpec ||
                heightMeasureSpec != mOldHeightMeasureSpec) {
            // first clears the measured dimension flag
            mPrivateFlags &= ~MEASURED_DIMENSION_SET;
            if (ViewDebug.TRACE_HIERARCHY) {
                ViewDebug.trace(this, ViewDebug.HierarchyTraceType.ON_MEASURE);
            }
            // measure ourselves, this should set the measured dimension flag back
            onMeasure(widthMeasureSpec, heightMeasureSpec);
            // flag not set, setMeasuredDimension() was not invoked, we raise
            // an exception to warn the developer
            if ((mPrivateFlags & MEASURED_DIMENSION_SET) != MEASURED_DIMENSION_SET) {
                throw new IllegalStateException("onMeasure() did not set the"
                        + " measured dimension by calling"
                        + " setMeasuredDimension()");
            }
            mPrivateFlags |= LAYOUT_REQUIRED;
        }
        mOldWidthMeasureSpec = widthMeasureSpec;
        mOldHeightMeasureSpec = heightMeasureSpec;
    }
    {% endhighlight %}

第二步。
 在随后的layout方法中，会判断这个标记。如果这个标记为true。
那么就一定会调用onLayout.
onLayout调用后清理LAYOUT_REQUIRED标记。
layout调用之后，会清理掉FORCE_LAYOUT标记。

    {% highlight java  %}
  @SuppressWarnings({"unchecked"})
   public void layout(int l, int t, int r, int b) {
       int oldL = mLeft;
       int oldT = mTop;
       int oldB = mBottom;
       int oldR = mRight;
       boolean changed = setFrame(l, t, r, b);
       if (changed || (mPrivateFlags & LAYOUT_REQUIRED) == LAYOUT_REQUIRED) {
           if (ViewDebug.TRACE_HIERARCHY) {
               ViewDebug.trace(this, ViewDebug.HierarchyTraceType.ON_LAYOUT);
           }
           onLayout(changed, l, t, r, b);
           mPrivateFlags &= ~LAYOUT_REQUIRED;
           ListenerInfo li = mListenerInfo;
           if (li != null && li.mOnLayoutChangeListeners != null) {
               ArrayList<OnLayoutChangeListener> listenersCopy =
                       (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
               int numListeners = listenersCopy.size();
               for (int i = 0; i < numListeners; ++i) {
                   listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
               }
           }
       }
       mPrivateFlags &= ~FORCE_LAYOUT;
   }
    {% endhighlight %}

当然在上述过程中，影响到了兄弟或者是父亲View的大小， 那么也兄弟或者是父亲View也会调用layout/onLayout。
不管其是否已经调用requestLayout。如果说指定的MeasureSpec为此也发生了变化，
那么measure/onMeasure也会被调用。

通过上述分析发现，只要调用了requestlayout, 那么measure和onMeasure，以及layout，onlayout，draw onDraw都会被调用。

#### 总结

requestLayout：当view确定自身已经不再适合现有的区域时，该view本身调用这个方法要求parent view重新调用他的onMeasure onLayout来对重新设置自己位置。

特别的当view的layoutparameter发生改变，并且它的值还没能应用到view上，这时候适合调用这个方法。