---
layout: post
author: mxn
titile: 深入了解android view绘制机制
category: 技术博文
tag: android
---

这篇文章讲解View的工作原理以及自定义View的方法

### LayoutInflater

先来看一下LayoutInflater的基本用法吧，它的用法非常简单，首先需要获取到LayoutInflater的实例，有两种方法可以获取到，第一种写法如下：

    {% highlight java  %}
    LayoutInflater layoutInflater = LayoutInflater.from(context);
    {% endhighlight %}

当然，还有另外一种写法也可以完成同样的效果：

    {% highlight java  %}
    LayoutInflater layoutInflater = (LayoutInflater) context
        .getSystemService(Context.LAYOUT_INFLATER_SERVICE);

        {% endhighlight %}

其实第一种就是第二种的简单写法，只是Android给我们做了一下封装而已。得到了LayoutInflater的实例之后就可以调用它的inflate()方法来加载布局了，如下所示：
layoutInflater.inflate(resourceId, root);

inflate()方法一般接收两个参数，第一个参数就是要加载的布局id，第二个参数是指给该布局的外部再嵌套一层父布局，如果不需要就直接传null。这样就成功成功创建了一个布局的实例，之后再将它添加到指定的位置就可以显示出来了。

接下来我们就从源码的角度上看一看LayoutInflater到底是如何工作的。

<!-- more -->

不管你是使用的哪个inflate()方法的重载，最终都会辗转调用到LayoutInflater的如下代码中：

    {% highlight java  %}
public View inflate(XmlPullParser parser, ViewGroup root, boolean attachToRoot) {
    synchronized (mConstructorArgs) {
        final AttributeSet attrs = Xml.asAttributeSet(parser);
        mConstructorArgs[0] = mContext;
        View result = root;
        try {
            int type;
            while ((type = parser.next()) != XmlPullParser.START_TAG &&
                    type != XmlPullParser.END_DOCUMENT) {
            }
            if (type != XmlPullParser.START_TAG) {
                throw new InflateException(parser.getPositionDescription()
                        + ": No start tag found!");
            }
            final String name = parser.getName();
            if (TAG_MERGE.equals(name)) {
                if (root == null || !attachToRoot) {
                    throw new InflateException("merge can be used only with a valid "
                            + "ViewGroup root and attachToRoot=true");
                }
                rInflate(parser, root, attrs);
            } else {
                View temp = createViewFromTag(name, attrs);
                ViewGroup.LayoutParams params = null;
                if (root != null) {
                    params = root.generateLayoutParams(attrs);
                    if (!attachToRoot) {
                        temp.setLayoutParams(params);
                    }
                }
                rInflate(parser, temp, attrs);
                if (root != null && attachToRoot) {
                    root.addView(temp, params);
                }
                if (root == null || !attachToRoot) {
                    result = temp;
                }
            }
        } catch (XmlPullParserException e) {
            InflateException ex = new InflateException(e.getMessage());
            ex.initCause(e);
            throw ex;
        } catch (IOException e) {
            InflateException ex = new InflateException(
                    parser.getPositionDescription()
                    + ": " + e.getMessage());
            ex.initCause(e);
            throw ex;
        }
        return result;
    }
}
    {% endhighlight %}

从这里我们就可以清楚地看出，LayoutInflater其实就是使用Android提供的pull解析方式来解析布局文件的。
调用了createViewFromTag()这个方法，并把节点名和参数传了进去。看到这个方法名，我们就应该能猜到，它是用于根据节点名来创建View对象的。
确实如此，在createViewFromTag()方法的内部又会去调用createView()方法，然后使用反射的方式创建出View的实例并返回。
当然，这里只是创建出了一个根布局的实例而已，接下来会调用rInflate()方法来循环遍历这个根布局下的子元素，代码如下所示：

    {% highlight java  %}
private void rInflate(XmlPullParser parser, View parent, final AttributeSet attrs)
        throws XmlPullParserException, IOException {
    final int depth = parser.getDepth();
    int type;
    while (((type = parser.next()) != XmlPullParser.END_TAG ||
            parser.getDepth() > depth) && type != XmlPullParser.END_DOCUMENT) {
        if (type != XmlPullParser.START_TAG) {
            continue;
        }
        final String name = parser.getName();
        if (TAG_REQUEST_FOCUS.equals(name)) {
            parseRequestFocus(parser, parent);
        } else if (TAG_INCLUDE.equals(name)) {
            if (parser.getDepth() == 0) {
                throw new InflateException("<include /> cannot be the root element");
            }
            parseInclude(parser, parent, attrs);
        } else if (TAG_MERGE.equals(name)) {
            throw new InflateException("<merge /> must be the root element");
        } else {
            final View view = createViewFromTag(name, attrs);
            final ViewGroup viewGroup = (ViewGroup) parent;
            final ViewGroup.LayoutParams params = viewGroup.generateLayoutParams(attrs);
            rInflate(parser, view, attrs);
            viewGroup.addView(view, params);
        }
    }
    parent.onFinishInflate();
}
        {% endhighlight %}

可以看到，同样是createViewFromTag()方法来创建View的实例，然后还会递归调用rInflate()方法来查找这个View下的子元素，每次递归完成后则将这个View添加到父布局当中。
这样的话，把整个布局文件都解析完成后就形成了一个完整的DOM结构，最终会把最顶层的根布局返回，至此inflate()过程全部结束。

```注意:通过inflate导入的根布局，不管你将根布局的layout_width和layout_height
的值修改成多少，都不会有任何效果的，因为这两个值现在已经完全失去了作用。平时我们经常使用layout_width和layout_height来设置View的大小，并且一直都能正常工作，就好像这两个属性确实是用于设置View的大小的。
而实际上则不然，它们其实是用于设置View在布局中的大小的，也就是说，首先View必须存在于一个布局中，之后如果将layout_width设置成match_parent表示让View的宽度填充满布局，如果设置成wrap_content表示让View的宽度刚好可以包含其内容，
如果设置成具体的数值则View的宽度会变成相应的数值。这也是为什么这两个属性叫作layout_width和layout_height，而不是width和height。
很明显通过inflate导入的根布局不存在于任何布局当中，所以layout_width和layout_height这两个属性理所当然没有任何作用。
在setContentView()方法中，Android会自动在布局文件的最外层再嵌套一个FrameLayout，所以layout_width和layout_height属性才会有效果。```


### view 绘制流程

任何一个视图都不可能凭空突然出现在屏幕上，它们都是要经过非常科学的绘制流程后才能显示出来的。
每一个视图的绘制过程都必须经历三个最主要的阶段，即onMeasure()、onLayout()和onDraw()，下面我们逐个对这三个阶段展开进行探讨。

measure是测量的意思，那么onMeasure()方法顾名思义就是用于测量视图的大小的。View系统的绘制流程会从ViewRoot的performTraversals()方法中开始，在其内部调用View的measure()方法。

```注意:窗口的顶层视图的父视图是使用一个ViewRoot对象来描述的，也就是说，当前正在处理的视图容器的成员变量mParent指向的是一个ViewRoot对象```

 ViewRoot.performTraversals这个函数定义在文件frameworks/base/core/java/android/view/ViewRoot.java中。
 代码如下：

    {% highlight java  %}
public final class ViewRoot extends Handler implements ViewParent,
        View.AttachInfo.Callbacks {
    ......
    private void performTraversals() {
        ......
        // cache mView since it is used so much below...
        final View host = mView;
        ......
        final boolean didLayout = mLayoutRequested;
        ......
        if (didLayout) {
            ......
            host.layout(0, 0, host.mMeasuredWidth, host.mMeasuredHeight);
            ......
            if ((host.mPrivateFlags & View.REQUEST_TRANSPARENT_REGIONS) != 0) {
                // start out transparent
                // TODO: AVOID THAT CALL BY CACHING THE RESULT?
                host.getLocationInWindow(mTmpLocation);
                mTransparentRegion.set(mTmpLocation[0], mTmpLocation[1],
                        mTmpLocation[0] + host.mRight - host.mLeft,
                        mTmpLocation[1] + host.mBottom - host.mTop);
                host.gatherTransparentRegion(mTransparentRegion);
                ......
                if (!mTransparentRegion.equals(mPreviousTransparentRegion)) {
                    mPreviousTransparentRegion.set(mTransparentRegion);
                    // reconfigure window manager
                    try {
                        sWindowSession.setTransparentRegion(mWindow, mTransparentRegion);
                    } catch (RemoteException e) {
                    }
                }
            }
            ......
        }
        ......
        boolean cancelDraw = attachInfo.mTreeObserver.dispatchOnPreDraw();
        if (!cancelDraw && !newSurface) {
            ......
            draw(fullRedrawNeeded);
            ......
        }
    }
    ......
}
            {% endhighlight %}

measure()方法接收两个参数，widthMeasureSpec和heightMeasureSpec，这两个值分别用于确定视图的宽度和高度的规格和大小。

MeasureSpec的值由specSize和specMode共同组成的，其中specSize记录的是大小，specMode记录的是规格。specMode一共有三种类型，如下所示：

1. EXACTLY

表示父视图希望子视图的大小应该是由specSize的值来决定的，系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

2. AT_MOST

表示子视图最多只能是specSize中指定的大小，开发人员应该尽可能小得去设置这个视图，并且保证不会超过specSize。系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

3. UNSPECIFIED

表示开发人员可以将视图按照自己的意愿设置成任意的大小，没有任何限制。这种情况比较少见，不太会用到。

widthMeasureSpec和heightMeasureSpec这两个值又是从哪里得到的呢？通常情况下，这两个值都是由父视图经过计算后传递给子视图的，说明父视图会在一定程度上决定子视图的大小。
但是最外层的根视图，它的widthMeasureSpec和heightMeasureSpec又是从哪里得到的呢？这就需要去分析ViewRoot中的源码了，观察performTraversals()方法可以发现如下代码：

    {% highlight java  %}
childWidthMeasureSpec = getRootMeasureSpec(desiredWindowWidth, lp.width);
childHeightMeasureSpec = getRootMeasureSpec(desiredWindowHeight, lp.height);
    {% endhighlight %}

可以看到，这里调用了getRootMeasureSpec()方法去获取widthMeasureSpec和heightMeasureSpec的值，注意方法中传入的参数，其中lp.width和lp.height在创建ViewGroup实例的时候就被赋值了，
它们都等于MATCH_PARENT。然后看下getRootMeasureSpec()方法中的代码，如下所示：

    {% highlight java  %}
private int getRootMeasureSpec(int windowSize, int rootDimension) {
    int measureSpec;
    switch (rootDimension) {
    case ViewGroup.LayoutParams.MATCH_PARENT:
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
        break;
    case ViewGroup.LayoutParams.WRAP_CONTENT:
        measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
        break;
    default:
        measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
        break;
    }
    return measureSpec;
}
    {% endhighlight %}

可以看到，这里使用了MeasureSpec.makeMeasureSpec()方法来组装一个MeasureSpec，当rootDimension参数等于MATCH_PARENT的时候，MeasureSpec的specMode就等于EXACTLY，当rootDimension等于WRAP_CONTENT的时候，MeasureSpec的specMode就等于AT_MOST。并且MATCH_PARENT和WRAP_CONTENT时的specSize都是等于windowSize的，也就意味着根视图总是会充满全屏的。

介绍了这么多MeasureSpec相关的内容，接下来我们看下View的measure()方法里面的代码吧，如下所示：

    {% highlight java  %}
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
    if ((mPrivateFlags & FORCE_LAYOUT) == FORCE_LAYOUT ||
            widthMeasureSpec != mOldWidthMeasureSpec ||
            heightMeasureSpec != mOldHeightMeasureSpec) {
        mPrivateFlags &= ~MEASURED_DIMENSION_SET;
        if (ViewDebug.TRACE_HIERARCHY) {
            ViewDebug.trace(this, ViewDebug.HierarchyTraceType.ON_MEASURE);
        }
        onMeasure(widthMeasureSpec, heightMeasureSpec);
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

注意观察，measure()这个方法是final的，因此我们无法在子类中去重写这个方法，说明Android是不允许我们改变View的measure框架的。然后在调用了onMeasure()
方法，这里才是真正去测量并设置View大小的地方。

    {% highlight java  %}
 protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
        {% endhighlight %}


默认会调用getDefaultSize()方法来获取视图的大小，如下所示：