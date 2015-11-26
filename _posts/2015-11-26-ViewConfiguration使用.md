---
layout: post
author: mxn
titile: ViewConfiguration使用
category: 技术博文
tag: android
---

### ViewConfiguration类

ViewConfiguration是系统中关于视图的各种特性的常量记录对象。其中包含各种基础数据,在编写自定义控件时会用经常用到。

下面介绍几个常用的方法：

1.在可滑动的控件中用于区别单击子控件和滑动操作的一个伐值。

    {% highlight java %}
mTouchSlop = configuration.getScaledTouchSlop();
   {% endhighlight %}

2.用于设置最小加速率和最大速率.

    {% highlight java %}
mMinimumVelocity = configuration.getScaledMinimumFlingVelocity();
mMaximumVelocity = configuration.getScaledMaximumFlingVelocity();
   {% endhighlight %}

3.滚动距离

    {% highlight java %}
mOverscrollDistance = configuration.getScaledOverscrollDistance();
   {% endhighlight %}
   
4.fling距离

    {% highlight java %}
mOverflingDistance = configuration.getScaledOverflingDistance();
   {% endhighlight %}
 
5.摩擦力，用来计算减速度

    {% highlight java %}
mDeceleration = computeDeceleration(ViewConfiguration.getScrollFriction());
   {% endhighlight %}
   

ViewConfiguration的常量总结：

    {% highlight java %}
/**    
  * 包含了方法和标准的常量用来设置UI的超时、大小和距离    
  */
 public class ViewConfiguration {     
     // 设定水平滚动条的宽度和垂直滚动条的高度，单位是像素px     
     private static final int SCROLL_BAR_SIZE = 10;     
           
     //定义滚动条逐渐消失的时间，单位是毫秒     
     private static final int SCROLL_BAR_FADE_DURATION = 250;     
           
     // 默认的滚动条多少秒之后消失，单位是毫秒     
     private static final int SCROLL_BAR_DEFAULT_DELAY = 300;     
           
     // 定义边缘地方褪色的长度     
     private static final int FADING_EDGE_LENGTH = 12;     
           
     //定义子控件按下状态的持续事件     
     private static final int PRESSED_STATE_DURATION = 125;     
               
     //定义一个按下状态转变成长按状态的转变时间     
     private static final int LONG_PRESS_TIMEOUT = 500;     
               
     //定义用户在按住适当按钮，弹出全局的对话框的持续时间     
     private static final int GLOBAL_ACTIONS_KEY_TIMEOUT = 500;     
               
     //定义一个touch事件中是点击事件还是一个滑动事件所需的时间，如果用户在这个时间之内滑动，那么就认为是一个点击事件     
     private static final int TAP_TIMEOUT = 115;     
               
     /**    
      * Defines the duration in milliseconds we will wait to see if a touch event     
      * is a jump tap. If the user does not complete the jump tap within this interval, it is    
      * considered to be a tap.     
      */
     //定义一个touch事件时候是一个点击事件。如果用户在这个时间内没有完成这个点击，那么就认为是一个点击事件     
     private static final int JUMP_TAP_TIMEOUT = 500;     
           
     //定义双击事件的间隔时间     
     private static final int DOUBLE_TAP_TIMEOUT = 300;     
               
     //定义一个缩放控制反馈到用户界面的时间     
     private static final int ZOOM_CONTROLS_TIMEOUT = 3000;     
           
     /**    
      * Inset in pixels to look for touchable content when the user touches the edge of the screen    
      */
     private static final int EDGE_SLOP = 12;     
               
     /**    
      * Distance a touch can wander before we think the user is scrolling in pixels    
      */
     private static final int TOUCH_SLOP = 16;     
               
     /**    
      * Distance a touch can wander before we think the user is attempting a paged scroll    
      * (in dips)    
      */
     private static final int PAGING_TOUCH_SLOP = TOUCH_SLOP * 2;     
               
     /**    
      * Distance between the first touch and second touch to still be considered a double tap    
      */
     private static final int DOUBLE_TAP_SLOP = 100;     
               
     /**    
      * Distance a touch needs to be outside of a window's bounds for it to    
      * count as outside for purposes of dismissing the window.    
      */
     private static final int WINDOW_TOUCH_SLOP = 16;     
           
    //用来初始化fling的最小速度，单位是每秒多少像素     
     private static final int MINIMUM_FLING_VELOCITY = 50;     
               
     //用来初始化fling的最大速度，单位是每秒多少像素     
     private static final int MAXIMUM_FLING_VELOCITY = 4000;     
           
     //视图绘图缓存的最大尺寸，以字节表示。在ARGB888格式下，这个尺寸应至少等于屏幕的大小     
     @Deprecated     
     private static final int MAXIMUM_DRAWING_CACHE_SIZE = 320 * 480 * 4; // HVGA screen, ARGB8888     
           
     //flings和scrolls摩擦力度大小的系数     
     private static float SCROLL_FRICTION = 0.015f;     
           
     /**    
      * Max distance to over scroll for edge effects    
      */
     private static final int OVERSCROLL_DISTANCE = 0;     
           
     /**    
      * Max distance to over fling for edge effects    
      */
     private static final int OVERFLING_DISTANCE = 4;     
           
 }
      {% endhighlight %}
   
      
### Configuration类

Configuration类虽然和ViewConfiguration类名字之差一个“view”,但是二者并无继承关系，也没有共同的父类。
Configuration类是专门用来描述手机设备上的配置信息。这些配置信息包括用户特定的配置项，也包括系统的动态设备配置。




