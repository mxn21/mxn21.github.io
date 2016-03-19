---
layout: post
author: mxn
titile: SurfaceView浅析
category: 技术博文
tag: android
---

### Surface

Surface对应了一块屏幕缓冲区，每个window对应一个Surface，任何View都要画在Surface的Canvas上,传统的view共享一块屏幕缓冲区，
所有的绘制必须在UI线程中进行。
关于Surface有如下特点：  

1、通过Surface（因为Surface是句柄）就可以获得原生缓冲器以及其中的内容.
2、原始缓冲区（a raw buffer）是用于保存当前窗口的像素数据的.
3、Surface中有一个Canvas成员，专门用于画图的.

可以认为Android中的Surface就是一个用来画图形（graphics）或图像（image）的地方。通常画图是在一个Canvas对象上面进行的，
由此，可以推知一个Surface对象中应该包含有一个Canvas（画布）对象。

### SurfaceView

SurfaceView继承自View，但是SurfaceView却有自己的Surface，SurfaceView的源码：

    {% highlight java %} 
if (mWindow == null) {    
         mWindow = new MyWindow(this);    
         mLayout.type = mWindowType;    
         mLayout.gravity = Gravity.LEFT|Gravity.TOP;    
         mSession.addWithoutInputChannel(mWindow, mWindow.mSeq, mLayout,    
         mVisible ? VISIBLE : GONE, mContentInsets);     
   }   
   {% endhighlight %} 

每个SurfaceView创建的时候都会创建一个MyWindow，new MyWindow(this)中的this正是SurfaceView自身，
因此将SurfaceView和window绑定在一起，因为一个window对应一个Surface，因此SurfaceView也就内嵌了一个自己的Surface，
可以认为SurfaceView就是展示Surface中数据的地方，是用来控制Surface中View的位置和尺寸的。

传统View及其派生类的更新只能在UI线程，然而UI线程还同时处理其他交互逻辑，这就无法保证View更新的速度和帧率了，
而SurfaceView可以用独立的线程进行绘制，因此可以提供更高的帧率，例如游戏，摄像头取景等场景就比较适合SurfaceView来实现。

总结一下SurfaceView和view的区别：

SurfaceView是从View基类中派生出来的显示类，直接子类有GLSurfaceView和VideoView，可以看出GL和视频播放以及Camera摄像头一般均使用SurfaceView。
SurfaceView和View最本质的区别在于，surfaceView是在一个新起的单独线程中可以重新绘制画面而View必须在UI的主线程中更新画面。 
那么在UI的主线程中更新画面可能会引发问题，比如你更新画面的时间过长，那么你的主UI线程会被你正在画的函数阻塞。那么将无法响应按键，触屏等消息。 
当使用surfaceView 由于是在新的线程中更新画面所以不会阻塞你的UI主线程。


### SurfaceHolder

SurfaceHolder是一个接口，其作用就像一个关于Surface的监听器，提供访问和控制SurfaceView内嵌的Surface相关的方法。
它通过三个回调方法，让我们可以感知到Surface的创建、销毁或者改变。

在SurfaceView中有一个方法getHolder，可以很方便地获得SurfaceView内嵌的Surface所对应的监听器接口SurfaceHolder。

SurfaceHolder还提供了几个重要的方法：

1、abstract void addCallback(SurfaceHolder.Callbackcallback)：为SurfaceHolder添加一个SurfaceHolder.Callback回调接口。
 
2、abstract Canvas lockCanvas()：获取一个Canvas对象，并锁定之。所得到的Canvas对象，其实就是Surface中一个成员。
 
3、abstract Canvas lockCanvas(Rect  dirty)：同上。但只锁定dirty所指定的矩形区域，因此效率更高。

4、abstract void unlockCanvasAndPost(Canvas  canvas)：当修改Surface中的数据完成后，释放同步锁，并提交改变，然后将新的数据进行展示，
同时Surface中相关数据会被丢失。
 
2、3、4中的同步锁机制的目的，就是为了在绘制的过程中，Surface中的数据不会被改变。lockCanvas是为了防止同一时刻多个线程对同一canvas写入。


### SurfaceHolder.Callback

SurfaceHolder.Callback主要是当底层的Surface被创建、销毁或者改变时提供回调通知，由于绘制必须在Surface被创建后才能进行，
因此SurfaceHolder.Callback中的surfaceCreated 和surfaceDestroyed 就成了绘图处理代码的边界。

SurfaceHolder.Callback中定义了三个接口方法：
 
1、abstract void surfaceChanged(SurfaceHolder holder, int format, int width, int height)：当surface发生任何结构性的变化时
（格式或者大小），该方法就会被立即调用。
 
2、abstract void surfaceCreated(SurfaceHolder holder)：当surface对象创建后，该方法就会被立即调用。
 
3、abstract void  surfaceDestroyed(SurfaceHolder holder)：当surface对象在将要销毁前，该方法会被立即调用。








