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

