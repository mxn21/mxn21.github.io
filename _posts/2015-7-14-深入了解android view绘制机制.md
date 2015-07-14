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

#### 一. onMeasure()


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

    {% highlight java  %}
 public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }
    {% endhighlight %}

这里传入的measureSpec是一直从measure()方法中传递过来的。然后调用MeasureSpec.getMode()方法可以解析出specMode，调用MeasureSpec.getSize()方法可以解析出specSize。接下来进行判断，如果specMode等于AT_MOST或EXACTLY就返回specSize，这也是系统默认的行为。
之后会在onMeasure()方法中调用setMeasuredDimension()方法来设定测量出的大小，这样一次measure过程就结束了。

当然，一个界面的展示可能会涉及到很多次的measure，因为一个布局中一般都会包含多个子视图，每个视图都需要经历一次measure过程。
ViewGroup中定义了一个measureChildren()方法来去测量子视图的大小，如下所示：

    {% highlight java  %}
protected void measureChildren(int widthMeasureSpec, int heightMeasureSpec) {
    final int size = mChildrenCount;
    final View[] children = mChildren;
    for (int i = 0; i < size; ++i) {
        final View child = children[i];
        if ((child.mViewFlags & VISIBILITY_MASK) != GONE) {
            measureChild(child, widthMeasureSpec, heightMeasureSpec);
        }
    }
}

    {% endhighlight %}

    这里首先会去遍历当前布局下的所有子视图，然后逐个调用measureChild()方法来测量相应子视图的大小，如下所示：

    {% highlight java  %}

    protected void measureChild(View child, int parentWidthMeasureSpec,
            int parentHeightMeasureSpec) {
        final LayoutParams lp = child.getLayoutParams();
        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom, lp.height);
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }

     {% endhighlight %}

可以看到，在第4行和第6行分别调用了getChildMeasureSpec()方法来去计算子视图的MeasureSpec，计算的依据就是布局文件中定义的MATCH_PARENT、WRAP_CONTENT等值，这个方法的内部细节就不再贴出。然后在第8行调用子视图的measure()方法，并把计算出的MeasureSpec传递进去，之后的流程就和前面所介绍的一样了。
当然，onMeasure()方法是可以重写的，也就是说，如果你不想使用系统默认的测量方式，可以按照自己的意愿进行定制，比如：


    {% highlight java  %}

public class MyView extends View {

    ......

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(200, 200);
    }

}

     {% endhighlight %}

这样的话就把View默认的测量流程覆盖掉了，不管在布局文件中定义MyView这个视图的大小是多少，最终在界面上显示的大小都将会是200*200。
需要注意的是，在setMeasuredDimension()方法调用之后，我们才能使用getMeasuredWidth()和getMeasuredHeight()来获取视图测量出的宽高，以此之前调用这两个方法得到的值都会是0。
由此可见，视图大小的控制是由父视图、布局文件、以及视图本身共同完成的，父视图会提供给子视图参考的大小，而开发人员可以在XML文件中指定视图的大小，然后视图本身会对最终的大小进行拍板。
到此为止，我们就把视图绘制流程的第一阶段分析完了。


#### 二. onLayout()

measure过程结束后，视图的大小就已经测量好了，接下来就是layout的过程了。正如其名字所描述的一样，这个方法是用于给视图进行布局的，也就是确定视图的位置。
ViewRoot的performTraversals()方法会在measure结束后继续执行，并调用View的layout()方法来执行此过程，如下所示：
host.layout(0, 0, host.mMeasuredWidth, host.mMeasuredHeight);

layout()方法接收四个参数，分别代表着左、上、右、下的坐标，当然这个坐标是相对于当前视图的父视图而言的。可以看到，这里还把刚才测量出的宽度和高度传到了layout()方法中。
那么我们来看下layout()方法中的代码是什么样的吧，如下所示：

    {% highlight java  %}
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
        if (mOnLayoutChangeListeners != null) {
            ArrayList<OnLayoutChangeListener> listenersCopy =
                    (ArrayList<OnLayoutChangeListener>) mOnLayoutChangeListeners.clone();
            int numListeners = listenersCopy.size();
            for (int i = 0; i < numListeners; ++i) {
                listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
            }
        }
    }
    mPrivateFlags &= ~FORCE_LAYOUT;
}
    {% endhighlight %}

在layout()方法中，首先会调用setFrame()方法来判断视图的大小是否发生过变化，以确定有没有必要对当前的视图进行重绘，同时还会在这里把传递过来的四个参数分别赋值给mLeft、mTop、mRight和mBottom这几个变量。
接下来会在第11行调用onLayout()方法。进入onLayout()方法，View中的onLayout()方法就是一个空方法，因为onLayout()过程是为了确定视图在布局中所在的位置，而这个操作应该是由布局来完成的，即父视图决定子视图的显示位置。
既然如此，我们来看下ViewGroup中的onLayout()方法是怎么写的吧，代码如下：

    {% highlight java  %}
@Override
protected abstract void onLayout(boolean changed, int l, int t, int r, int b);

    {% endhighlight %}

可以看到，ViewGroup中的onLayout()方法竟然是一个抽象方法，这就意味着所有ViewGroup的子类都必须重写这个方法。没错，像LinearLayout、RelativeLayout等布局，都是重写了这个方法，
然后在内部按照各自的规则对子视图进行布局的。由于LinearLayout和RelativeLayout的布局规则都比较复杂，就不单独拿出来进行分析了，这里我们尝试自定义一个布局，借此来更深刻地理解onLayout()的过程。

自定义的这个布局目标很简单，只要能够包含一个子视图，并且让子视图正常显示出来就可以了。那么就给这个布局起名叫做SimpleLayout吧，代码如下所示：

    {% highlight java  %}
    public class SimpleLayout extends ViewGroup {

        public SimpleLayout(Context context, AttributeSet attrs) {
            super(context, attrs);
        }

        @Override
        protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
            super.onMeasure(widthMeasureSpec, heightMeasureSpec);
            if (getChildCount() > 0) {
                View childView = getChildAt(0);
                measureChild(childView, widthMeasureSpec, heightMeasureSpec);
            }
        }

        @Override
        protected void onLayout(boolean changed, int l, int t, int r, int b) {
            if (getChildCount() > 0) {
                View childView = getChildAt(0);
                childView.layout(0, 0, childView.getMeasuredWidth(), childView.getMeasuredHeight());
            }
        }

    }
    {% endhighlight %}

代码非常的简单，我们来看下具体的逻辑吧。你已经知道，onMeasure()方法会在onLayout()方法之前调用，因此这里在onMeasure()方法中判断SimpleLayout中是否有包含一个子视图，如果有的话就调用measureChild()方法来测量出子视图的大小。
接着在onLayout()方法中同样判断SimpleLayout是否有包含一个子视图，然后调用这个子视图的layout()方法来确定它在SimpleLayout布局中的位置，这里传入的四个参数依次是0、0、childView.getMeasuredWidth()和childView.getMeasuredHeight()，分别代表着子视图在SimpleLayout中左上右下四个点的坐标。其中，调用childView.getMeasuredWidth()和childView.getMeasuredHeight()方法得到的值就是在onMeasure()方法中测量出的宽和高。

如果你想改变ImageView显示的位置，只需要改变childView.layout()方法的四个参数就行了。

在onLayout()过程结束后，我们就可以调用getWidth()方法和getHeight()方法来获取视图的宽高了。说到这里，我相信很多朋友长久以来都会有一个疑问，getWidth()方法和getMeasureWidth()方法到底有什么区别呢？它们的值好像永远都是相同的。其实它们的值之所以会相同基本都是因为布局设计者的编码习惯非常好，实际上它们之间的差别还是挺大的。

首先getMeasureWidth()方法在measure()过程结束后就可以获取到了，而getWidth()方法要在layout()过程结束后才能获取到。另外，getMeasureWidth()方法中的值是通过setMeasuredDimension()方法来进行设置的，而getWidth()方法中的值则是通过视图右边的坐标减去左边的坐标计算出来的。

观察SimpleLayout中onLayout()方法的代码，这里给子视图的layout()方法传入的四个参数分别是0、0、childView.getMeasuredWidth()和childView.getMeasuredHeight()，因此getWidth()方法得到的值就是childView.getMeasuredWidth() - 0 = childView.getMeasuredWidth() ，所以此时getWidth()方法和getMeasuredWidth() 得到的值就是相同的，但如果你将onLayout()方法中的代码进行如下修改：


    {% highlight java  %}
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        if (getChildCount() > 0) {
            View childView = getChildAt(0);
            childView.layout(0, 0, 200, 200);
        }
    }

        {% endhighlight %}

这样getWidth()方法得到的值就是200 - 0 = 200，不会再和getMeasuredWidth()的值相同了。当然这种做法充分不尊重measure()过程计算出的结果，通常情况下是不推荐这么写的。getHeight()与getMeasureHeight()方法之间的关系同上，就不再重复分析了。
到此为止，我们把视图绘制流程的第二阶段也分析完了。

#### 三. onDraw()

measure和layout的过程都结束后，接下来就进入到draw的过程了。同样，根据名字你就能够判断出，在这里才真正地开始对视图进行绘制。ViewRoot中的代码会继续执行并创建出一个Canvas对象，然后调用View的draw()方法来执行具体的绘制工作。draw()方法内部的绘制过程总共可以分为六步，
其中第二步和第五步在一般情况下很少用到，因此这里我们只分析简化后的绘制过程。代码如下所示：

    {% highlight java  %}
    public void draw(Canvas canvas) {
        if (ViewDebug.TRACE_HIERARCHY) {
            ViewDebug.trace(this, ViewDebug.HierarchyTraceType.DRAW);
        }
        final int privateFlags = mPrivateFlags;
        final boolean dirtyOpaque = (privateFlags & DIRTY_MASK) == DIRTY_OPAQUE &&
                (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
        mPrivateFlags = (privateFlags & ~DIRTY_MASK) | DRAWN;
        // Step 1, draw the background, if needed
        int saveCount;
        if (!dirtyOpaque) {
            final Drawable background = mBGDrawable;
            if (background != null) {
                final int scrollX = mScrollX;
                final int scrollY = mScrollY;
                if (mBackgroundSizeChanged) {
                    background.setBounds(0, 0,  mRight - mLeft, mBottom - mTop);
                    mBackgroundSizeChanged = false;
                }
                if ((scrollX | scrollY) == 0) {
                    background.draw(canvas);
                } else {
                    canvas.translate(scrollX, scrollY);
                    background.draw(canvas);
                    canvas.translate(-scrollX, -scrollY);
                }
            }
        }
        final int viewFlags = mViewFlags;
        boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
        boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
        if (!verticalEdges && !horizontalEdges) {
            // Step 3, draw the content
            if (!dirtyOpaque) onDraw(canvas);
            // Step 4, draw the children
            dispatchDraw(canvas);
            // Step 6, draw decorations (scrollbars)
            onDrawScrollBars(canvas);
            // we're done...
            return;
        }
    }
     {% endhighlight %}

可以看到，第一步是从第9行代码开始的，这一步的作用是对视图的背景进行绘制。这里会先得到一个mBGDrawable对象，然后根据layout过程确定的视图位置来设置背景的绘制区域，之后再调用Drawable的draw()方法来完成背景的绘制工作。那么这个mBGDrawable对象是从哪里来的呢？其实就是在XML中通过android:background属性设置的图片或颜色。当然你也可以在代码中通过setBackgroundColor()、setBackgroundResource()等方法进行赋值。
接下来的第三步是在第34行执行的，这一步的作用是对视图的内容进行绘制。可以看到，这里去调用了一下onDraw()方法，那么onDraw()方法里又写了什么代码呢？进去一看你会发现，原来又是个空方法啊。其实也可以理解，因为每个视图的内容部分肯定都是各不相同的，这部分的功能交给子类来去实现也是理所当然的。

第三步完成之后紧接着会执行第四步，这一步的作用是对当前视图的所有子视图进行绘制。但如果当前的视图没有子视图，那么也就不需要进行绘制了。因此你会发现View中的dispatchDraw()方法又是一个空方法，而ViewGroup的dispatchDraw()方法中就会有具体的绘制代码。

以上都执行完后就会进入到第六步，也是最后一步，这一步的作用是对视图的滚动条进行绘制。那么你可能会奇怪，当前的视图又不一定是ListView或者ScrollView，为什么要绘制滚动条呢？其实不管是Button也好，TextView也好，任何一个视图都是有滚动条的，只是一般情况下我们都没有让它显示出来而已。绘制滚动条的代码逻辑也比较复杂，这里就不再贴出来了，因为我们的重点是第三步过程。

通过以上流程分析，相信大家已经知道，View是不会帮我们绘制内容部分的，因此需要每个视图根据想要展示的内容来自行绘制。如果你去观察TextView、ImageView等类的源码，你会发现它们都有重写onDraw()这个方法，并且在里面执行了相当不少的绘制逻辑。绘制的方式主要是借助Canvas这个类，它会作为参数传入到onDraw()方法中，供给每个视图使用。Canvas这个类的用法非常丰富，基本可以把它当成一块画布，在上面绘制任意的东西，那么我们就来尝试一下吧。

这里简单起见，我只是创建一个非常简单的视图，并且用Canvas随便绘制了一点东西，代码如下所示：

    {% highlight java  %}
public class MyView extends View {

    private Paint mPaint;

    public MyView(Context context, AttributeSet attrs) {
        super(context, attrs);
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        mPaint.setColor(Color.YELLOW);
        canvas.drawRect(0, 0, getWidth(), getHeight(), mPaint);
        mPaint.setColor(Color.BLUE);
        mPaint.setTextSize(20);
        String text = "Hello View";
        canvas.drawText(text, 0, getHeight() / 2, mPaint);
    }
}

     {% endhighlight %}