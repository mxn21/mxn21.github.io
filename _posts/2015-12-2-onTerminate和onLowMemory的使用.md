---
layout: post
author: mxn
titile: onTerminate和onLowMemory的使用
category: 技术博文
tag: android
---

关于Application类有以下公开方法：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img136.jpg)

其中最常用的就是onCreaet()、onTerminate()以及onLowMemory()方法，onCreaet()它会在这个类实例化的时候调用（这个地方需要十分注意，
它只会在程序第一次启动的时候调用，不是每次都调用，因为Application类会在创建包的时候初始化），onTerminate()会在app关闭的时候调用，
onLowMemory()是在内存过低的情况下调用的。那onTerminate()是干什么用的呢？网上介绍说是在程序终结的时候调用，从下面的实例中也是这样用的。
不过在Google的开发文档以及很多书籍对这个类的用法很少提及，但实际开发情况是如何的呢。

<!-- more -->

### onTerminate的使用

onTerminate()会在app关闭的时候调用,但是就像onDestroy()一样，不能保证一定会被调用。所以最好不要依赖这个方法做重要的处理，
这个方法最多可以用来销毁一写对象，清除一下缓存，但是也并不能保证一定会清除掉，其他操作，例如想在程序结束保存数据，用这个方法明显是错误的。

使用方法如下：

清除http连接：

	{% highlight java  %}
 public void onTerminate()
  {
    super.onTerminate();
    shutdownHttpClient();
  }
  
   private void shutdownHttpClient()
    {
      if ((this.httpClient != null) && (this.httpClient.getConnectionManager() != null)) {
        this.httpClient.getConnectionManager().shutdown();
      }
    }
     {% endhighlight %}
     
清除缓存、日志和地图

	{% highlight java  %}
 public void onTerminate()
  {
    if (this.mBMapMan != null)
    {
      this.mBMapMan.destroy();
      this.mBMapMan = null;
    }
    super.onTerminate();
    new Thread()
    {
      public void run()
      {
        App.delete(new File(Environment.getExternalStorageDirectory() + "/xxx/Diary/"));
      }
    }.start();
  }
  
       {% endhighlight %}
    
清除ActivityLifecycleCallbacks的注册
   
       	{% highlight java  %}
    @Override
    public void onTerminate() {
        unregisterActivityLifecycleCallbacks(mActivityWatcher);
        super.onTerminate();
    }
          {% endhighlight %}
        
          
### onLowMemory

onLowMemory()在内存比较紧张时,根据优先级把后台程序杀死时,系统回调他,它用在14之前,14之后就出现了onTrimMemory()

源码如下：

       	{% highlight java  %}
    public void onLowMemory() {
        Object[] callbacks = collectComponentCallbacks();
        if (callbacks != null) {
            for (int i=0; i<callbacks.length; i++) {
                ((ComponentCallbacks)callbacks[i]).onLowMemory();
            }
        }
    }

    public void onTrimMemory(int level) {
        Object[] callbacks = collectComponentCallbacks();
        if (callbacks != null) {
            for (int i=0; i<callbacks.length; i++) {
                Object c = callbacks[i];
                if (c instanceof ComponentCallbacks2) {
                    ((ComponentCallbacks2)c).onTrimMemory(level);
                }
            }
        }
    }
          {% endhighlight %}

onTrimMemory(int level)是根据级别不同做不同的操作:

TRIM_MEMORY_COMPLETE:
系统处于低内存的运行状态中如果系统现在没有内存回收你的应用将会第一个被杀掉. 你必须释放掉所有非关键的资源从而恢复应用的状态.
TRIM_MEMORY_MODERATE

 
系统处于低内存的运行状态中并且你的应用处于缓存应用列表的中级阶段. 如果系运行内存收到限制, 你的应用有被杀掉的风险.
TRIM_MEMORY_BACKGROUND:
    系统处于低内存的运行状态中并且你的应用处于缓存应用列表的初级阶段.  虽然你的应用不会处于被杀的高风险中, 但是系统已经开始清除缓存列表中的其它应用, 所以你必须释放资源使你的应用继续存留在列表中以便用户再次回到你的应用时能快速恢复进行使用.
TRIM_MEMORY_UI_HIDDEN
     这个过程显示到用户界面,提示占用内存比较大的应用和ui即将被释放,ui不可见
TRIM_MEMORY_RUNNING_CRITICAL
    应用处于运行状态但是系统已经把大多数缓存应用杀掉了, 你必须释放掉不是非常关键的资源, 如果系统不能回收足够的运行内存, 系统会清除所有缓存应用并且会把正在活动的应用杀掉.
TRIM_MEMORY_RUNNING_LOW
    应用处于运行状态并且不会被杀掉, 设备可以使用的内存非常低, 可以把不用的资源释放一些提高性能(会直接影响程序的性能)
TRIM_MEMORY_RUNNING_MODERATE
   应用处于运行状态并且不会被杀掉, 设备使用的内存比较低, 系统级会杀掉一些其它的缓存应用.


OnLowMemory()和OnTrimMemory()的比较
1，OnLowMemory被回调时，已经没有后台进程；而onTrimMemory被回调时，还有后台进程。
2，OnLowMemory是在最后一个后台进程被杀时调用，一般情况是low memory killer 杀进程后触发；而OnTrimMemory的触发更频繁，每次计算进程优先级时，只要满足条件，都会触发。
3，通过一键清理后，OnLowMemory不会被触发，而OnTrimMemory会被触发一次。