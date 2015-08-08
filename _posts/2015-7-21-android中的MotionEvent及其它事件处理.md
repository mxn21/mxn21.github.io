---
layout: post
author: mxn
titile: android中的MotionEvent及其它事件处理
category: 技术博文
tag: android
---

### MotionEvent

MotionEvent即一个触摸事件。对屏幕的几乎所有操作都会触发事件，如点击、放开、滑动等。

不同的事件在MotionEvent中有不同的id，我们可以根据event.getAction() & MotionEvent.ACTION_MASK的结果来判断是何种事件。

有如下事件是我们要用到的：

MotionEvent.ACTION_DOWN：在第一个点被按下时触发。

MotionEvent.ACTION_UP:当屏幕上唯一的点被放开时触发。

MotionEvent.ACTION_CANCEL:当你的手指（或者其它）移动出范围的时候，当前的手势被中断，会触发这个事件。

在设计设置页面的滑动开关时，如果不监听ACTION_CANCEL，在滑动到中间时，如果你手指上下移动，就是移动到开关控件之外，
则此时会触发ACTION_CANCEL，而不是ACTION_UP，造成开关的按钮停顿在中间位置。
意思就是，当用户保持按下操作，并从你的控件转移到外层控件时，会触发ACTION_CANCEL。

MotionEvent.ACTION_POINTER_DOWN:当屏幕上已经有一个点被按住，此时再按下其他点时触发。

MotionEvent.ACTION_POINTER_UP:当屏幕上有多个点被按住，松开其中一个点时触发（即非最后一个点被放开时）。

MotionEvent.ACTION_MOVE：当有点在屏幕上移动时触发。值得注意的是，由于它的灵敏度很高，
而我们的手指又不可能完全静止（即使我们感觉不到移动，但其实我们的手指也在不停地抖动），
所以实际的情况是，基本上只要有点在屏幕上，此事件就会一直不停地被触发。

举例子来说：当我们放一个食指到屏幕上时，触发ACTION_DOWN事件；
再放一个中指到屏幕上，触发ACTION_POINTER_DOWN事件；此时再把食指或中指放开，都会触发ACTION_POINTER_UP事件；
再放开最后一个手指，触发ACTION_UP事件；而同时在整个过程中，ACTION_MOVE事件会一直不停地被触发。

event.getX(index)和event.getY(index)可以获取到指定index点的坐标，所以当屏幕上有两个点的时候，我们用如下方法来获取两点间的距离：

    {% highlight java  %}
    private float spacing(MotionEvent event) {
         float x = event.getX(0) - event.getX(1);
         float y = event.getY(0) - event.getY(1);
        return FloatMath.sqrt(x * x + y * y);
    }
    {% endhighlight %}

<!-- more -->

### MotionEvent 中getAction()与getActionMasked()的区别

首先看代码:

    {% highlight java  %}
    public static final int ACTION_MASK = 0xff;

    public final int getAction() {
        return mAction;
    }

    public final int getActionMasked() {
    return mAction & ACTION_MASK;
    }
    {% endhighlight %}

他们有什么区别呢？如果mAction的值是在0x00到0xff之间的话。getAction()返回的值，和getActionMasked()的返回的值是一样的。
那什么时候返回的值是不一样的呢？即当mAction值大于0xff时，那什么时候会大于0xff呢？
这就是是当有多点触控时。

mAction的低8位即0x00到0xff用来表示动作的类型信息。例如：MotionEvent#ACTION_DOWN的值是 0,即0x00。MotionEvent#ACTION_UP的值是 1，即0x01。
但是，我们知道Android是支持多点触控的，那么怎么知道这个一个MotionEvent是哪一个触控点触发的呢？那么就还需要MotionEvent带有触控点索引信息。
Android的解决方案时在；mAction的第二个8位中存储。例如，如果mAction的值是0x0000，则表示是第一个触控点的ACTION_DOWN操作。
如果mAction的值是0x0100呢，则表示是第二个触控点的ACTION_DOWN操作。第三个的ACTION_DOWN呢？相信你可以推出来是0x0200。
总而言之，mAction时的低8位（也就是0-7位）是动作类型信息。mAction的8-15位呢，是触控点的索引信息。（即表示是哪一个触控点的事件）。

不过通常我们都是以不同的字段来存储不同的信息。但是在计算机内部他们还是变成了0,1。计算机始终还是以位来存储信息的。如果我们熟悉以位为基本单位来理解信息的存储,
对于理解android中的很多变量是很有帮助的。因为其中的很多东西使用的这样的节约内存的技巧。用一个字段即可以节约内存。又可以方便提高处理速度。

### 手势坐标介绍

下面解释Android系统坐标及触摸MotionEvent e1和e2、速度velocityX、velocityY等值

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img29.jpg)

（1）MotionEvent中 e1是手指第一次按上屏幕的起点，e2是抬起手指离开屏幕的终点，根据上图Android屏幕坐标系可知：

手指向右滑动，终点（e2）在起点（e1）的右侧，有e2.getX() - e1.getX() 大于0
手指向左滑动，终点（e2）在起点（e1）的左侧，有e2.getX() - e1.getX() 小于0
手指向下滑动，终点（e2）在起点（e1）的下侧，有e2.getY() - e1.getY() 大于0
手指向上滑动，终点（e2）在起点（e1）的上侧，有e2.getY() - e1.getY() 小于0

 （2）onScroll(MotionEvent e1, MotionEvent e2, float distanceX, float distanceY)

distanceX，是前后两次call的X距离，不是e2与e1的水平距离

distanceY，是前后两次call的Y距离，不是e2与e1的垂直距离

具体数值的方向，请详见上图（中）

 （3）onFling(MotionEvent e1, MotionEvent e2, float velocityX, float velocityY)

velocityX，是X轴的每秒速度

velocityY，是Y轴的每秒速度

具体数值的方向，请详见上图（右）

仔细观察可以发现：velocityX、velocityY的方向与distanceX、distanceY方向正好相反.

