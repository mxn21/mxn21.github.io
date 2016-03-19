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






