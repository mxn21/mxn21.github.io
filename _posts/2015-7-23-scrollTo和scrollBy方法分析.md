---
layout: post
author: mxn
titile: scrollTo和scrollBy方法分析
category: 技术博文
tag: android
---

android系统手机屏幕的左上角为坐标系，同时y轴方向与笛卡尔坐标系的y轴方向想反。通过提供的api如getLeft , getTop, getBottom,
getRight可以获得控件在parent中的相对位置。同时，也可以获得控件在屏幕中的绝对位置。

当我们编写一些自定义的滑动控件时，会用到一些api如scrollTo(),scrollBy(),getScrollX(), getScrollY()。
由于常常会对函数getScrollX(), getScrollY()返回的值的含义产生混淆，尤其是正负关系。本文将讲解他们的用法。

注意：调用View的scrollTo()和scrollBy()是用于滑动View中的内容，而不是把某个View的位置进行改变。如果想改变莫个View在屏幕中的位置，可以使用如下的方法。
调用public void offsetLeftAndRight(int offset)用于左右移动方法或public void offsetTopAndBottom(int offset)用于上下移动。
如：button.offsetLeftAndRignt(300)表示将button控件向左移动300个像素。


### scrollTo(int x, int y)

scrollTo(int x, int y) 是将View中内容滑动到相应的位置，参考的坐标系原点为parent View的左上角。
调用scrollTo(100, 0)表示将View中的内容移动到x = 100， y = 0的位置，如下图所示。
注意，图中黄色矩形区域表示的是一个parent View，绿色虚线矩形为parent view中的内容。一般情况下两者的大小一致，
本文为了显示方便，将虚线框画小了一点。图中的黄色区域的位置始终不变，发生位置变化的是显示的内容。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img38.png)

同理，scrollTo(0, 100)的效果如下图所示：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img39.png)

scrollTo(100, 100)的效果图如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img40.png)

若函数中参数为负值，则子View的移动方向将相反.

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img41.png)

### scrollBy(int x, int y)

scrollBy(int x, int y)其实是对scrollTo的包装，移动的是相当位置。 scrollTo(int x, int y)的源码和scrollBy(int x, int y)源码如下所示.

    {% highlight java  %}

public void scrollTo(int x, int y) {
    if (mScrollX != x || mScrollY != y) {
        int oldX = mScrollX;
        int oldY = mScrollY;
        mScrollX = x;
        mScrollY = y;
        invalidateParentCaches();
        onScrollChanged(mScrollX, mScrollY, oldX, oldY);
        if (!awakenScrollBars()) {
            postInvalidateOnAnimation();
        }
    }
}

public void scrollBy(int x, int y) { scrollTo(mScrollX + x, mScrollY + y); }

    {% endhighlight %}


可见，mScrollX和mScrollY是View类中专门用于记录滑动位置的变量。这两个函数最终调用onScrollChanged()函数.

理解了scrollTo(int x, int y)和scrollBy(int x, int y)的用法，就不难理解getScrollX() 和getScrollY()。这两个函数的源码如下所示：


    {% highlight java  %}

public final int getScrollX() {
    return mScrollX;
}

public final int getScrollY() {
    return mScrollY;
}

    {% endhighlight %}



