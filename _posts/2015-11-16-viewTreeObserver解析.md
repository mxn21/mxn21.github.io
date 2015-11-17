---
layout: post
author: mxn
titile: ViewTreeObserver解析
category: 技术博文
tag: android
---

ViewTreeObserver是view事件的观察者。这个观察者用来监听视图树，会监听视图树发生全局变化时发出的通知。这里指的全局
事件包括而且不局限在以下几个：整个视图树的布局变化，开始绘制视图，触摸模式改变等等。

ViewTreeObserver是不能被应用程序实例化的，因为它是由视图提供的，通过view.getViewTreeObserver()获取。

ViewTreeObserver提供了view的很多种监听，每一种监听在ViewTreeObserver中都有一个内部类接口来定义。查看源码可以看到
一共有11个内部类，全部保存在CopyOnWriteArrayList数组中，通过ViewTreeObserver.addXXXListener()来添加这些监听，
所以这11个接口对应了11个add方法，但是有一些方法是还没有开放的，用@hide隐藏了。我目前测试的源码在compileSdkVersion 22的基础上。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img128.png)

<!-- more -->

### api的概述

下面介绍其中常用的几个内部类：

    {% highlight java %}
interface  ViewTreeObserver.OnGlobalFocusChangeListener        
//当在一个视图树中的焦点状态发生改变时，所要调用的回调函数的接口类
 
interface  ViewTreeObserver.OnGlobalLayoutListener
//当在一个视图树中全局布局发生改变或者视图树中的某个视图的可视状态发生改变时，所要调用的回调函数的接口类
 
interface  ViewTreeObserver.OnPreDrawListener
//当一个视图树将要绘制时，所要调用的回调函数的接口类
 
interface  ViewTreeObserver.OnScrollChangedListener
//当一个视图树中的一些组件发生滚动时，所要调用的回调函数的接口类
 
interface  ViewTreeObserver.OnTouchModeChangeListener
//当一个视图树的触摸模式发生改变时，所要调用的回调函数的接口类
    {% endhighlight %}
    
    
他们对应的add方法如下：

    {% highlight java %}
/**注册一个回调函数，当在一个视图树中的焦点状态发生改变时调用这个回调函数。
　 * 参数 listener    将要被添加的回调函数
　 *异常 IllegalStateException       如果isAlive() 返回false
　 */
　public void addOnGlobalFocusChangeListener (ViewTreeObserver.OnGlobalFocusChangeListener listener)
     
 
　/**注册一个回调函数，当在一个视图树中全局布局发生改变或者视图树中的某个视图的可视状态发生改变时调用这个回调函数。
　 *参数 listener    将要被添加的回调函数
　 *异常 IllegalStateException       如果isAlive() 返回false
　 */
　public void addOnGlobalLayoutListener (ViewTreeObserver.OnGlobalLayoutListener listener)
　　
 
　　
　/**注册一个回调函数，当一个视图树将要绘制时调用这个回调函数。
　 *参数 listener    将要被添加的回调函数
　 *异常 IllegalStateException       如果isAlive() 返回false
　　*/
　public void addOnPreDrawListener (ViewTreeObserver.OnPreDrawListener listener)
 
　   
　/**注册一个回调函数，当一个视图发生滚动时调用这个回调函数。
　 *参数 listener    将要被添加的回调函数
　 *异常 IllegalStateException       如果isAlive() 返回false
　　*/
　public void addOnScrollChangedListener (ViewTreeObserver.OnScrollChangedListener listener)  
 
　
　/**注册一个回调函数，当一个触摸模式发生改变时调用这个回调函数。
　 *参数 listener    将要被添加的回调函数
　 *异常 IllegalStateException       如果isAlive() 返回false
　 */
　public void addOnTouchModeChangeListener (ViewTreeObserver.OnTouchModeChangeListener listener)

    {% endhighlight %}


还可以调用remove方法删除监听

他们对应的add方法如下

    {% highlight java %}
    
/**移除之前已经注册的全局布局回调函数。
　 *参数 victim 将要被移除的回调函数
　 *异常 IllegalStateException       如果isAlive() 返回false   
　 */
　public void removeGlobalOnLayoutListener (ViewTreeObserver.OnGlobalLayoutListener victim)
　　
　/**移除之前已经注册的焦点改变回调函数。
　 *参数 victim 将要被移除的回调函数
　　*异常 IllegalStateException       如果isAlive() 返回false 
　 */
　public void removeOnGlobalFocusChangeListener (ViewTreeObserver.OnGlobalFocusChangeListener victim)
　　
　/**移除之前已经注册的预绘制回调函数。
　　*参数 victim 将要被移除的回调函数
　 *异常 IllegalStateException       如果isAlive() 返回false  
　 */
　public void removeOnPreDrawListener (ViewTreeObserver.OnPreDrawListener victim)
　　
　/**移除之前已经注册的滚动改变回调函数。
　 *参数 victim 将要被移除的回调函数
　　*异常 IllegalStateException       如果isAlive() 返回false 
　　*/
　public void removeOnScrollChangedListener (ViewTreeObserver.OnScrollChangedListener victim)
　
　/**移除之前已经注册的触摸模式改变回调函数
　　*参数 victim 将要被移除的回调函数
　　*异常 　IllegalStateException       如果isAlive() 返回false
　 */
　public void removeOnTouchModeChangeListener (ViewTreeObserver.OnTouchModeChangeListener victim)
    {% endhighlight %}
    
    
其他的常用方法：

    {% highlight java %}
　//当整个布局发生改变时通知相应的注册监听器。如果你强制对视图布局或者在一个没有附加到一个窗口的视图的层次结构或者在GONE状态下，它可以被手动的调用
　public final void dispatchOnGlobalLayout ()
    
　/**当一个视图树将要绘制时通知相应的注册监听器。如果这个监听器返回true，则这个绘制将被取消并重新计划。如果你强制对视图布局或者在一个没有附加到一个窗口的视图的层次结构或者在一个GONE状态下，它可以被手动的调用
　 *返回值  当前绘制能够取消并重新计划则返回true，否则返回false。
　 */
　public final boolean dispatchOnPreDraw ()
 
　/**指示当前的ViewTreeObserver是否可用(alive)。当observer不可用时，任何方法的调用（除了这个方法）都将抛出一个异常。如果一个应用程序保持和ViewTreeObserver一个历时较长的引用，它应该总是需要在调用别的方法之前去检测这个方法的返回值。
　　*返回值 但这个对象可用则返回true，否则返回false   
　 */
　public boolean isAlive ()

    {% endhighlight %}
    
### 用法举例

#### 用于测量view的宽高

先写一个自定义view如下：

    {% highlight java %}
public class MyImageView extends ImageView {
    public MyImageView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    public MyImageView(Context context) {
        super(context);
    }
    @Override
    protected void onMeasure(int widthMeasureSpec,int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        Log.d("===MyImageView","onMeasure 我被调用了"+System.currentTimeMillis());
    }
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        Log.d("===MyImageView", "onDraw 我被调用了"+System.currentTimeMillis());
    }
}

    {% endhighlight %}
    
MainActivity布局文件如下：

    {% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
    <com.mxn.soul.demo.MyImageView
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/imageview"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/test" />

    {% endhighlight %}    
    
MainActivity中的代码:
   
       {% highlight java %}
     @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            final MyImageView myImageView = (MyImageView) findViewById(R.id.imageview);
            Log.d("===MainActivity", "onCreate执行完毕..myImageView " +
                    "height:" + myImageView.getMeasuredHeight() + "  ,width:" + myImageView.getMeasuredWidth());
        }
    
    {% endhighlight %}  
    
输入结果如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img129.png)
    
首先应该知道的是在MainActivity的onCreate中调用myImageView.getHeight()的值是为0的，因为在onCreate中还没有调用onLayout()，
具体原因参考，我在之前的日志：[深入了解android view绘制机制](http://souly.cn/技术博文/2015/07/14/深入了解android%20view绘制机制/)  .

简单的说就是，在onLayout()过程结束后，我们就可以调用getWidth()方法和getHeight()方法来获取视图的宽高了，getWidth()方法和getMeasureWidth()
方法到底有什么区别呢？它们的值好像永远都是相同的。其实它们的值之所以会相同基本都是因为布局设计者的编码习惯非常好，
实际上它们之间的差别还是挺大的。首先getMeasureWidth()方法在measure()过程结束后就可以获取到了，而getWidth()方法要在layout()
过程结束后才能获取到。另外，getMeasureWidth()方法中的值是通过setMeasuredDimension()方法来进行设置的，
而getWidth()方法中的值则是通过视图右边的坐标减去左边的坐标计算出来的。

一般有三种方法可以在onCreate中计算控件宽高：

第一种方法：

    {% highlight java %}
 @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        final MyImageView myImageView = (MyImageView) findViewById(R.id.imageview);
        int w = View.MeasureSpec.makeMeasureSpec(0,View.MeasureSpec.UNSPECIFIED);
        int h = View.MeasureSpec.makeMeasureSpec(0,View.MeasureSpec.UNSPECIFIED);
        myImageView.measure(w, h);
        Log.d("===MainActivity", "onCreate执行完毕..myImageView " +
                "height:" + myImageView.getMeasuredHeight() + "  ,width:" + myImageView.getMeasuredWidth());
    }

    {% endhighlight %}  

输出结果如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img130.png)

可以看到在onCreate手动调用了onMeasure，这样就可以通过getMeasuredHeight()取得宽度和高度了。


