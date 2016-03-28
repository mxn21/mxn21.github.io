---
layout: post
author: mxn
titile: SurfaceView浅析
category: 技术博文
tag: android
---

在一般情况下，应用程序的View都是在相同的GUI线程中绘制的。这个主应用程序线程同时也用来处理所有的用户交互（例如按钮单击或者文本输入）。
对于一个View的onDraw()方法，不能够满足将其移动到后台线程中去。因为从后台线程修改一个GUI元素会被显式地禁止的。
当需要快速地更新View的UI，或者当前渲染代码阻塞GUI线程的时间过长的时候，SurfaceView就是解决上述问题的最佳选择。
SurfaceView封装了一个Surface对象，而不是Canvas。因为Surface可以使用后台线程绘制。对于那些资源敏感的操作，
或者那些要求快速更新或者高速帧率的地方，例如使用3D图形，创建游戏，或者实时预览摄像头，这一点特别有用。

下面分别介绍Surface相关的几个类：

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



### 基本用法

我们知道使用自定义view可以做一些简单的动画效果。它通过不断循环的执行View.onDraw方法，每次执行都对内部显示的图形做一些调整，我们假设 
onDraw方法每秒执行20次，这样就会形成一个20帧的补间动画效果。但是现实情况是你无法简单的控制View.onDraw的执行帧数，
这边说的执行帧数是指每秒View.onDraw方法被执行多少次，这是为什么呢？首先我们知道，onDraw方法是由系统帮我们调用的，
我们是通过调用View的invalidate方法通知系统需要重新绘制View，然后它就会调用View.onDraw方法。这些都是由系统帮我们实现的，
所以我们很难精确去定义View.onDraw的执行帧数，这个就是为什么我们这边要了解SurfaceView了，它能弥补View的一些不足。

首先我们先写一个自定义View实现动画效果，AnimateViewActivity.java：

    {% highlight java %} 
public class AnimateViewActivity extends Activity {  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(new AnimateView(this));
    }  
    class AnimateView extends View{  
        float radius = 10;  
        Paint paint;  
        public AnimateView(Context context) {  
            super(context);  
            paint = new Paint();  
            paint.setColor(Color.YELLOW);  
            paint.setStyle(Paint.Style.STROKE);  
        }  
        @Override  
        protected void onDraw(Canvas canvas) {  
            canvas.translate(200, 200);  
            canvas.drawCircle(0, 0, radius++, paint);            
            if(radius > 100){  
                radius = 10;  
            }  
            invalidate();//通过调用这个方法让系统自动刷新视图  
        }  
    }  
}  
   {% endhighlight %} 
   
运行上面的Activity，你将看到一个圆圈，它原始半径是10，然后不断的变大，直到达到100后又恢复到10，这样循环显示，
视觉效果上说你将看到一个逐渐变大的圆圈。效果如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img185.png)

它能做的只是简单的动画效果，具有一些局限性。首先你无法控制动画的显示速度，目前它是以最快的速度显示，但是当你要更快，
获取帧数更高的动画呢？因为View的帧数是由系统控制的，所以你没办法完成上面的操作。如果你需要编写一个游戏，它需要的帧数比较高，
那么View就无能为力了，因为它被设计出来时本来就不是用来处理一些高帧数显示的。你可以把View理解为一个经过系统优化的，
可以用来高效的执行一些帧数比较低动画的对象，它具有特定的使用场景，比如有一些帧数较低的游戏就可以使用它来完成：贪吃蛇、
俄罗斯方块、棋牌类等游戏，因为这些游戏执行的帧数都很低。但是如果是一些实时类的游戏，如 射击游戏、塔防游戏、RPG游戏等就
没办法使用View来做，因为它的帧数太低了，会导致动画执行不顺畅。所以我们需要一个能自己控制执行帧数的对象，SurfaceView因此诞生了。

那么SurfaceView怎么控制帧数的呢？SurfaceView允许其他线程(不是UI线程)绘制图形(使用Canvas)，根据它这个特性，
你就可以控制它的帧数，你如果让这个线程1秒执行50次绘制，那么最后显示的就是50帧，先看看代码：

    {% highlight java %} 
public class DemoSurfaceView extends SurfaceView  implements SurfaceHolder.Callback{  
    public DemoSurfaceView(Context context) {  
        super(context);  
        init(); //初始化,设置生命周期回调方法  
    }  
    private void init(){  
        SurfaceHolder holder = getHolder();  
        holder.addCallback(this); //设置Surface生命周期回调  
    }  
    @Override  
    public void surfaceChanged(SurfaceHolder holder, int format, int width,  
            int height) {  
    }  
    @Override  
    public void surfaceCreated(SurfaceHolder holder) {  
    }  
    @Override  
    public void surfaceDestroyed(SurfaceHolder holder) {  
    }  
}  
   {% endhighlight %} 

上面代码我们在SurfaceView的构造方法中执行了init初始化方法，在这个方法里，我们先获取SurfaceView里的 SurfaceHolder对象，
然后通过它设置Surface的生命周期回调方法，使用DemoSurfaceView类本身作为回调方法代理类。 surfaceCreated方法，
是当SurfaceView被显示时会调用的方法，所以你需要再这边开启绘制的线 程，surfaceDestroyed方法是当SurfaceView被隐藏会销毁时调用的方法，
在这里你可以关闭绘制的线程。上面的例子运行后什么也不显示，因为还没定义一个执行绘制的线程。下面我们修改下代码，使用一个线程绘制一个
逐渐变大的圆圈：

    {% highlight java %} 
public class DemoSurfaceView extends SurfaceView  implements SurfaceHolder.Callback{  
    LoopThread thread;  
    public DemoSurfaceView(Context context) {  
        super(context);  
        init(); //初始化,设置生命周期回调方法  
    }  
    private void init(){  
        SurfaceHolder holder = getHolder();  
        holder.addCallback(this); //设置Surface生命周期回调  
        thread = new LoopThread(holder, getContext());  
    }  
    @Override  
    public void surfaceChanged(SurfaceHolder holder, int format, int width,  
            int height) {  
    }  
    @Override  
    public void surfaceCreated(SurfaceHolder holder) {  
        thread.isRunning = true;  
        thread.start();  
    }  
    @Override  
    public void surfaceDestroyed(SurfaceHolder holder) {  
        thread.isRunning = false;  
        try {  
            thread.join();  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        }  
    }  
    /** 
     * 执行绘制的绘制线程 
     * @author Administrator 
     * 
     */  
    class LoopThread extends Thread{  
        SurfaceHolder surfaceHolder;  
        Context context;  
        boolean isRunning;  
        float radius = 10f;  
        Paint paint;  
        public LoopThread(SurfaceHolder surfaceHolder,Context context){  
            this.surfaceHolder = surfaceHolder;  
            this.context = context;  
            isRunning = false;  
            paint = new Paint();  
            paint.setColor(Color.YELLOW);  
            paint.setStyle(Paint.Style.STROKE);  
        }  
        @Override  
        public void run() {  
            Canvas c = null;  
            while(isRunning){  
                try{  
                    synchronized (surfaceHolder) {  
                        c = surfaceHolder.lockCanvas(null);  
                        doDraw(c);  
                        //通过它来控制帧数执行一次绘制后休息50ms  
                        Thread.sleep(50);  
                    }  
                } catch (InterruptedException e) {  
                    e.printStackTrace();  
                } finally {  
                    surfaceHolder.unlockCanvasAndPost(c);  
                }  
            }  
        }  
        public void doDraw(Canvas c){  
            //这个很重要，清屏操作，清楚掉上次绘制的残留图像  
            c.drawColor(Color.BLACK);  
            c.translate(200, 200);  
            c.drawCircle(0,0, radius++, paint);  
            if(radius > 100){  
                radius = 10f;  
            }  
        }  
    }  
}     
    {% endhighlight %} 
    
上面代码编写了一个使用SurfaceView制作的动画效果，它的效果跟上面自定义View的一样，但是这边的SurfaceView可以控制动画的帧数。
在SurfaceView中内置一个LoopThread线程，这个线程的作用就是用来绘制图形，在SurfaceView中实例化一个LoopThread实例，
一般这个操作会放在SurfaceView的构造方法中。然后通过在SurfaceView中的SurfaceHolder的生命周期回调方法中插入一些操作，
当Surface被创建时(SurfaceView显示在屏幕中时)，开启LoopThread执行绘制，LoopThread会一直刷新SurfaceView对象，
当SurfaceView被隐藏时就停止改线程释放资源。这边有几个地方要注意下：
                           
1.因为SurfaceView允许自定义的线程操作Surface对象执行绘制方法，而你可能同时定义多个线程执行绘制，所以当你获取
SurfaceHolder中的Canvas对象时记得加同步操作，避免两个不同的线程同时操作同一个Canvas对象，当操作完成后记得调用
SurfaceHolder.unlockCanvasAndPost方法释放掉Canvas锁。
                           
2.在调用doDraw执行绘制时，因为SurfaceView的特点，它会保留之前绘制的图形，所以你需要先清空掉上一次绘制时留下的图形。
(View则不会，它默认在调用View.onDraw方法时就自动清空掉视图里的东西)。
                           
3. 记得在回调方法：onSurfaceDestroyed方法里将后台执行绘制的LoopThread关闭，这里是使用join方法。这涉及到线程如何关闭的问题，
多数人建议是通过一个标志位：isRunning来判断线程是否该停止运行，如果你想关闭线程只需要将isRunning改成false即可，
线程会自动执行完run方法后退出。

#### SurfaceView设置透明背景的方法

将SurfaceView背景设置为透明其实很简单主要添加以下几句话就可以了.

在SurfaceView创建后设置一下下面的参数：

    {% highlight java %} 
setZOrderOnTop(true);
getHolder().setFormat(PixelFormat.TRANSLUCENT);
    {% endhighlight %} 

还有在draw方法中绘制背景颜色的时候以下面的方式进行绘制就可以实现SurfaceView的背景透明化,
将上面代码中的  c.drawColor(Color.BLACK);  改为

    {% highlight java %} 
canvas.drawColor(Color.TRANSPARENT,Mode.CLEAR);
    {% endhighlight %} 
    
    
### 双缓存(Double-buffer)与黑屏闪烁

1.每个SurfaceView 对象有两个独立的graphic buffer，官方SDK将它们称作"front buffer"和"back buffer"。

2.常规的"double-buffer"会这么做：每一帧的数据都被绘制到back buffer，然后back buffer的内容被持续翻转(flip)到front 
buffer；屏幕一直显示front buffer。但Android SurfaceView的"double-buffer"却是这么做的：在buffer A里绘制内容，
然后让屏幕显示buffer A; 下一个循环，在buffer B里绘制内容，然后让屏幕显示buffer B; 如此往复。于是，屏幕上显示的
内容依次来自buffer A, B, A, B,....这样看来，两个buffer其实没有主从的分别，与其称之为"front buffer""back buffer"，
不如称之为"buffer A""buffer B"。

3.Android中"double-buffer"的实现机制，可以很好地解释闪屏现象。在第一个"lockCanvas-drawCanvas-unlockCanvasAndPost
"循环中，更新的是buffer A的内容；到下一个"lockCanvas-drawCanvas-unlockCanvasAndPost"循环中，更新的是buffer B的内容。
如果buffer A与buffer B中某个buffer内容为空，当屏幕轮流显示它们时，就会出现画面黑屏闪烁现象。

#### 解决方法

出现黑屏是因为buffer A与buffer B中一者内容为空，而且为空的一方还被post到了屏幕。于是有两种解决思路：

1.不让空buffer出现：每次向一个buffer写完内容并post之后，顺便用这个buffer的内容填充另一个buffer。这样能保证两个
buffer的内容是同步的，缺点是做了无用功，耗费性能。

2.不post空buffer到屏幕：当准备更新内容时，先判断内容是否为空，只有非空时才启动"lockCanvas-drawCanvas-unlockCanvasAndPost"这个流程。

#### 双缓存的演示

我们写一个demo，效果是在屏幕上随机画一些彩色的点，每次绘制时不清除上一次的点，这些点是累加的。效果图如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img186.png)

整个屏幕是一个MyGameSurfaceView, main.xml文件包含MyGameSurfaceView,代码如下：

    {% highlight xml %} 
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
 android:layout_width="fill_parent"
 android:layout_height="fill_parent"
 android:orientation="vertical" >
 <TextView
     android:layout_width="fill_parent"
     android:layout_height="wrap_content"
     android:text="@string/hello" />
 <com.MyGame.MyGameSurfaceView
     android:id="@+id/myview1"
     android:layout_width="fill_parent"
     android:layout_height="fill_parent" />
</LinearLayout>
    {% endhighlight %} 

下面写main activity，我们这里叫MyGameActivity，并且在onResume()中调用myGameSurfaceView1.MyGameSurfaceView_OnResume()，
在onPause()中调用myGameSurfaceView1.MyGameSurfaceView_OnPause()，这两个方法用来启动和关闭后台的绘制进程。

    {% highlight java %} 
public class MyGameActivity extends Activity {
 MyGameSurfaceView myGameSurfaceView1;
 /** Called when the activity is first created. */
 @Override
 public void onCreate(Bundle savedInstanceState) {
     super.onCreate(savedInstanceState);
     setContentView(R.layout.main);
     myGameSurfaceView1 = (MyGameSurfaceView)findViewById(R.id.myview1);
 }
 @Override
 protected void onResume() {
  // TODO Auto-generated method stub
  super.onResume();
  myGameSurfaceView1.MyGameSurfaceView_OnResume();
 }
 @Override
 protected void onPause() {
  // TODO Auto-generated method stub
  super.onPause();
  myGameSurfaceView1.MyGameSurfaceView_OnPause();
 }
}
      {% endhighlight %}   

下面是MyGameThread.java文件，这是一个典型的后台线程的结构，需要注意的是run()方法中的parent.updateSurfaceView()，
它会调用MyGameSurfaceView里的updateSurfaceView()方法来进行绘制。

下面是MyGameSurfaceView.java文件，这是核心部分。updateSurfaceView()会在前面的线程里循环调用， 这个方法里又调用了
updateStates()和onDraw(canvas)分别用于更新状态和在屏幕绘制。注意updateSurfaceView()是在后台线程调用的，而不是UI线程。

    {% highlight java %} 
public class MyGameSurfaceView extends SurfaceView implements SurfaceHolder.Callback{
 SurfaceHolder surfaceHolder;
 MyGameThread myGameThread = null;
 private Paint paint = new Paint(Paint.ANTI_ALIAS_FLAG);
 Random random;
 public MyGameSurfaceView(Context context) {
  super(context);
  // TODO Auto-generated constructor stub
 }
 public MyGameSurfaceView(Context context, AttributeSet attrs) {
  super(context, attrs);
  // TODO Auto-generated constructor stub
 }
 public MyGameSurfaceView(Context context, AttributeSet attrs, int defStyle) {
  super(context, attrs, defStyle);
  // TODO Auto-generated constructor stub
 }
 @Override
 public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
  // TODO Auto-generated method stub
 }
 @Override
 public void surfaceCreated(SurfaceHolder holder) {
  // TODO Auto-generated method stub
 }
 @Override
 public void surfaceDestroyed(SurfaceHolder holder) {
  // TODO Auto-generated method stub
 }
 public void MyGameSurfaceView_OnResume(){
  random = new Random();
  surfaceHolder = getHolder();
  getHolder().addCallback(this);
  //Create and start background Thread
  myGameThread = new MyGameThread(this, 500);
  myGameThread.setRunning(true);
  myGameThread.start();
 }
 public void MyGameSurfaceView_OnPause(){
  //Kill the background Thread
  boolean retry = true;
  myGameThread.setRunning(false);
  while(retry){
   try {
    myGameThread.join();
    retry = false; 
   } catch (InterruptedException e) {
    e.printStackTrace(); 
   } 
  }
 }
 @Override
 protected void onDraw(Canvas canvas) {
  paint.setStyle(Paint.Style.STROKE);
     paint.setStrokeWidth(3);
     int w = canvas.getWidth();
     int h = canvas.getHeight();
     int x = random.nextInt(w-1);
     int y = random.nextInt(h-1);
     int r = random.nextInt(255);
     int g = random.nextInt(255);
     int b = random.nextInt(255);
     paint.setColor(0xff000000 + (r << 16) + (g << 8) + b);
     canvas.drawPoint(x, y, paint);
 }
 public void updateStates(){
  //Dummy method() to handle the States
 }
 public void updateSurfaceView(){
  //The function run in background thread, not ui thread.
  Canvas canvas = null;
  try{
   canvas = surfaceHolder.lockCanvas();
   synchronized (surfaceHolder) {
    updateStates();
    onDraw(canvas);
   }
  }finally{
   if(canvas != null){
    surfaceHolder.unlockCanvasAndPost(canvas);
   }
  } 
 }
}
     {% endhighlight %} 
     
这样我们的demo就完成了。如果你运行了上面的代码，你就会注意到屏幕在两个bitmap之间闪烁，这就是SurfaceView的double-buffer的原因：
当你在绘制Buffer A的时候，Buffer B正在被展示，然后你再画Buffer B，这时Buffer A被展示。因此，我们在Buffer A和Buffer B上是独立的
画了一些随机点，而不是同一个bitmap。这个特性可以解决一些显示性能的问题，但是在这里并不是我们想要的效果。

为了解决这个问题，我们需要在一个独立的bitmap上绘制，然后把这个bitmap绘制到canvas上。

下面我们修改MyGameSurfaceView.java文件如下：





     
### SurfaceView原理分析

SurfaceView原理分析参考如下：

[Android视图SurfaceView的实现原理分析](http://blog.csdn.net/luoshengyang/article/details/8661317)   

[Android SurfaceView 源码分析及使用](http://tech.youzan.com/surfaceview-sourcecode/)

[Graphics architecture](https://source.android.com/devices/graphics/architecture.html#activity)
    
