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

