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