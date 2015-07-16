---
layout: post
author: mxn
titile: android view状态及重绘流程分析
category: 技术博文
tag: android
---

### 视图状态

视图状态的种类非常多，一共有十几种类型，不过多数情况下我们只会使用到其中的几种，因此这里我们也就只去分析最常用的几种视图状态。

1. enabled

表示当前视图是否可用。可以调用setEnable()方法来改变视图的可用状态，传入true表示可用，传入false表示不可用。它们之间最大的区别在于，不可用的视图是无法响应onTouch事件的。

2. focused

表示当前视图是否获得到焦点。通常情况下有两种方法可以让视图获得焦点，即通过键盘的上下左右键切换视图，以及调用requestFocus()方法。而现在的Android手机几乎都没有键盘了，因此基本上只可以使用requestFocus()这个办法来让视图获得焦点了。而requestFocus()方法也不能保证一定可以让视图获得焦点，它会有一个布尔值的返回值，如果返回true说明获得焦点成功，返回false说明获得焦点失败。一般只有视图在focusable和focusable in touch mode同时成立的情况下才能成功获取焦点，比如说EditText。

3. window_focused

表示当前视图是否处于正在交互的窗口中，这个值由系统自动决定，应用程序不能进行改变。

4. selected

表示当前视图是否处于选中状态。一个界面当中可以有多个视图处于选中状态，调用setSelected()方法能够改变视图的选中状态，传入true表示选中，传入false表示未选中。

5. pressed

表示当前视图是否处于按下状态。可以调用setPressed()方法来对这一状态进行改变，传入true表示按下，传入false表示未按下。通常情况下这个状态都是由系统自动赋值的，但开发者也可以自己调用这个方法来进行改变。

我们可以在项目的drawable目录下创建一个selector文件，在这里配置每种状态下视图对应的背景图片。但是它的背景原理到底是怎样的呢？这就又要从源码的层次上进行分析了。

我们都知道，当手指按在视图上的时候，视图的状态就已经发生了变化，此时视图的pressed状态是true。每当视图的状态有发生改变的时候，就会回调View的drawableStateChanged()方法，代码如下所示：

    {% highlight java  %}
protected void drawableStateChanged() {
    Drawable d = mBGDrawable;
    if (d != null && d.isStateful()) {
        d.setState(getDrawableState());
    }
}
    {% endhighlight %}

在这里的第一步，首先是将mBGDrawable赋值给一个Drawable对象，那么这个mBGDrawable是什么呢？观察setBackgroundResource()方法中的代码，如下所示：

    {% highlight java  %}
public void setBackgroundResource(int resid) {
    if (resid != 0 && resid == mBackgroundResource) {
        return;
    }
    Drawable d= null;
    if (resid != 0) {
        d = mResources.getDrawable(resid);
    }
    setBackgroundDrawable(d);
    mBackgroundResource = resid;
}
      {% endhighlight %}

可以看到，在第7行调用了Resource的getDrawable()方法将resid转换成了一个Drawable对象，然后调用了setBackgroundDrawable()方法并将这个Drawable对象传入，在setBackgroundDrawable()方法中会将传入的Drawable对象赋值给mBGDrawable。
而我们在布局文件中通过android:background属性指定的selector文件，效果等同于调用setBackgroundResource()方法。也就是说drawableStateChanged()方法中的mBGDrawable对象其实就是我们指定的selector文件。

接下来在drawableStateChanged()方法的第4行调用了getDrawableState()方法来获取视图状态，代码如下所示：

    {% highlight java  %}
public final int[] getDrawableState() {
    if ((mDrawableState != null) && ((mPrivateFlags & DRAWABLE_STATE_DIRTY) == 0)) {
        return mDrawableState;
    } else {
        mDrawableState = onCreateDrawableState(0);
        mPrivateFlags &= ~DRAWABLE_STATE_DIRTY;
        return mDrawableState;
    }
}
     {% endhighlight %}

在这里首先会判断当前视图的状态是否发生了改变，如果没有改变就直接返回当前的视图状态，如果发生了改变就调用onCreateDrawableState()方法来获取最新的视图状态。视图的所有状态会以一个整型数组的形式返回。
在得到了视图状态的数组之后，就会调用Drawable的setState()方法来对状态进行更新，代码如下所示：

    {% highlight java  %}

public boolean setState(final int[] stateSet) {
    if (!Arrays.equals(mStateSet, stateSet)) {
        mStateSet = stateSet;
        return onStateChange(stateSet);
    }
    return false;
}
     {% endhighlight %}

这里会调用Arrays.equals()方法来判断视图状态的数组是否发生了变化，如果发生了变化则调用onStateChange()方法，否则就直接返回false。但你会发现，Drawable的onStateChange()方法中其实就只是简单返回了一个false，并没有任何的逻辑处理，这是为什么呢？这主要是因为mBGDrawable对象是通过一个selector文件创建出来的，而通过这种文件创建出来的Drawable对象其实都是一个StateListDrawable实例，
因此这里调用的onStateChange()方法实际上调用的是StateListDrawable中的onStateChange()方法:

    {% highlight java  %}
    @Override
    protected boolean onStateChange(int[] stateSet) {
        int idx = mStateListState.indexOfStateSet(stateSet);
        if (DEBUG) android.util.Log.i(TAG, "onStateChange " + this + " states "
                + Arrays.toString(stateSet) + " found " + idx);
        if (idx < 0) {
            idx = mStateListState.indexOfStateSet(StateSet.WILD_CARD);
        }
        if (selectDrawable(idx)) {
            return true;
        }
        return super.onStateChange(stateSet);
    }

         {% endhighlight %}


可以看到，这里会先调用indexOfStateSet()方法来找到当前视图状态所对应的Drawable资源下标，然后在第9行调用selectDrawable()方法并将下标传入，在这个方法中就会将视图的背景图设置为当前视图状态所对应的那张图片了。

那你可能会有疑问，在前面一篇文章中我们说到，任何一个视图的显示都要经过非常科学的绘制流程的，很显然，背景图的绘制是在draw()方法中完成的，那么为什么selectDrawable()方法能够控制背景图的改变呢？这就要研究一下视图重绘的流程了。


### 视图重绘

虽然视图会在Activity加载完成之后自动绘制到屏幕上，但是我们完全有理由在与Activity进行交互的时候要求动态更新视图，比如改变视图的状态、以及显示或隐藏某个控件等。那在这个时候，之前绘制出的视图其实就已经过期了，此时我们就应该对视图进行重绘。

调用视图的setVisibility()、setEnabled()、setSelected()等方法时都会导致视图重绘，而如果我们想要手动地强制让视图进行重绘，可以调用invalidate()方法来实现。当然了，setVisibility()、setEnabled()、setSelected()等方法的内部其实也是通过调用invalidate()方法来实现的，那么就让我们来看一看invalidate()方法的代码是什么样的吧。

View的源码中会有数个invalidate()方法的重载和一个invalidateDrawable()方法，当然它们的原理都是相同的，因此我们只分析其中一种，代码如下所示：

    {% highlight java  %}
    void invalidate(boolean invalidateCache) {
        if (ViewDebug.TRACE_HIERARCHY) {
            ViewDebug.trace(this, ViewDebug.HierarchyTraceType.INVALIDATE);
        }
        if (skipInvalidate()) {
            return;
        }
        if ((mPrivateFlags & (DRAWN | HAS_BOUNDS)) == (DRAWN | HAS_BOUNDS) ||
                (invalidateCache && (mPrivateFlags & DRAWING_CACHE_VALID) == DRAWING_CACHE_VALID) ||
                (mPrivateFlags & INVALIDATED) != INVALIDATED || isOpaque() != mLastIsOpaque) {
            mLastIsOpaque = isOpaque();
            mPrivateFlags &= ~DRAWN;
            mPrivateFlags |= DIRTY;
            if (invalidateCache) {
                mPrivateFlags |= INVALIDATED;
                mPrivateFlags &= ~DRAWING_CACHE_VALID;
            }
            final AttachInfo ai = mAttachInfo;
            final ViewParent p = mParent;
            if (!HardwareRenderer.RENDER_DIRTY_REGIONS) {
                if (p != null && ai != null && ai.mHardwareAccelerated) {
                    p.invalidateChild(this, null);
                    return;
                }
            }
            if (p != null && ai != null) {
                final Rect r = ai.mTmpInvalRect;
                r.set(0, 0, mRight - mLeft, mBottom - mTop);
                p.invalidateChild(this, r);
            }
        }
    }
     {% endhighlight %}

在这个方法中首先会调用skipInvalidate()方法来判断当前View是否需要重绘，判断的逻辑也比较简单，
如果View是不可见的且没有执行任何动画，就认为不需要重绘了。之后会进行透明度的判断，并给View添加一些标记位，
然后在第22和29行调用ViewParent的invalidateChild()方法，这里的ViewParent其实就是当前视图的父视图，
因此会调用到ViewGroup的invalidateChild()方法中，代码如下所示：

    {% highlight java  %}

    public final void invalidateChild(View child, final Rect dirty) {
        ViewParent parent = this;
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            final boolean drawAnimation = (child.mPrivateFlags & DRAW_ANIMATION) == DRAW_ANIMATION;
            if (dirty == null) {
                ......
            } else {
                ......
                do {
                    View view = null;
                    if (parent instanceof View) {
                        view = (View) parent;
                        if (view.mLayerType != LAYER_TYPE_NONE &&
                                view.getParent() instanceof View) {
                            final View grandParent = (View) view.getParent();
                            grandParent.mPrivateFlags |= INVALIDATED;
                            grandParent.mPrivateFlags &= ~DRAWING_CACHE_VALID;
                        }
                    }
                    if (drawAnimation) {
                        if (view != null) {
                            view.mPrivateFlags |= DRAW_ANIMATION;
                        } else if (parent instanceof ViewRootImpl) {
                            ((ViewRootImpl) parent).mIsAnimating = true;
                        }
                    }
                    if (view != null) {
                        if ((view.mViewFlags & FADING_EDGE_MASK) != 0 &&
                                view.getSolidColor() == 0) {
                            opaqueFlag = DIRTY;
                        }
                        if ((view.mPrivateFlags & DIRTY_MASK) != DIRTY) {
                            view.mPrivateFlags = (view.mPrivateFlags & ~DIRTY_MASK) | opaqueFlag;
                        }
                    }
                    parent = parent.invalidateChildInParent(location, dirty);
                    if (view != null) {
                        Matrix m = view.getMatrix();
                        if (!m.isIdentity()) {
                            RectF boundingRect = attachInfo.mTmpTransformRect;
                            boundingRect.set(dirty);
                            m.mapRect(boundingRect);
                            dirty.set((int) boundingRect.left, (int) boundingRect.top,
                                    (int) (boundingRect.right + 0.5f),
                                    (int) (boundingRect.bottom + 0.5f));
                        }
                    }
                } while (parent != null);
            }
        }
    }
     {% endhighlight %}

可以看到，这里在第10行进入了一个while循环，当ViewParent不等于空的时候就会一直循环下去。
在这个while循环当中会不断地获取当前布局的父布局，并调用它的invalidateChildInParent()方法，
在ViewGroup的invalidateChildInParent()方法中主要是来计算需要重绘的矩形区域，这里我们先不管它，当循环到最外层的根布局后，
就会调用ViewRoot的invalidateChildInParent()方法了，代码如下所示：

    {% highlight java  %}
    public ViewParent invalidateChildInParent(final int[] location, final Rect dirty) {
        invalidateChild(null, dirty);
        return null;
    }
    {% endhighlight %}

这里的代码非常简单，仅仅是去调用了invalidateChild()方法而已:

    {% highlight java  %}

public void invalidateChild(View child, Rect dirty) {
    checkThread();
    if (LOCAL_LOGV) Log.v(TAG, "Invalidate child: " + dirty);
    mDirty.union(dirty);
    if (!mWillDrawSoon) {
        scheduleTraversals();
    }
}  
    {% endhighlight %}