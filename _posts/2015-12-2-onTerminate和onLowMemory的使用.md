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
          
onLowMemory:Android系统提供了一些回调来通知当前应用的内存使用情况，通常来说，当所有的background应用都被kill掉的时候，forground
应用会收到onLowMemory()的回调。在这种情况下，需要尽快释放当前应用的非必须的内存资源，从而确保系统能够继续稳定运行。

OnTrimMemory:Android系统从4.0开始还提供了onTrimMemory()的回调，当系统内存达到某些条件的时候，所有正在运行的应用都会收到这个回调，
同时在这个回调里面会传递以下的参数，代表不同的内存使用情况，收到onTrimMemory()回调的时候，需要根据传递的参数类型进行判断，
合理的选择释放自身的一些内存占用，一方面可以提高系统的整体运行流畅度，另外也可以避免自己被系统判断为优先需要杀掉的应用。

* TRIM_MEMORY_UI_HIDDEN 
你的应用程序的所有UI界面被隐藏了，即用户点击了Home键或者Back键退出应用，导致应用的UI界面完全不可见。
这个时候应该释放一些不可见的时候非必须的资源.

当程序正在前台运行的时候，可能会接收到从onTrimMemory()中返回的下面的值之一：

* TRIM_MEMORY_RUNNING_CRITICAL
你的应用仍在运行，但是系统已经把LRU Cache中的大多数进程都已经杀死，因此你应该立即释放所有非必须的资源。如果系统不能回收到足够的RAM数量，
系统将会清除所有的LRU缓存中的进程，并且开始杀死那些之前被认为不应该杀死的进程，例如那个包含了一个运行态Service的进程。

* TRIM_MEMORY_RUNNING_LOW
你的应用正在运行且没有被列为可杀死的。但是设备正运行于更低内存的状态下，你应该释放不用的资源用来提升系统性能。

* TRIM_MEMORY_RUNNING_MODERATE
你的应用正在运行并且不会被列为可杀死的。但是设备此时正运行于低内存状态下，系统开始触发杀死LRU Cache中的Process的机制。

当应用进程退到后台正在被Cached的时候，可能会接收到从onTrimMemory()中返回的下面的值之一：

* TRIM_MEMORY_BACKGROUND
系统正运行于低内存状态并且你的进程正处于LRU缓存名单中最不容易杀掉的位置。尽管你的应用进程并不是处于被杀掉的高危险状态，系统可能已经开始杀掉LRU缓存中的其他进程了。你应该释放那些容易恢复的资源，以便于你的进程可以保留下来，这样当用户回退到你的应用的时候才能够迅速恢复。

* TRIM_MEMORY_MODERATE
系统正运行于低内存状态并且你的进程已经已经接近LRU名单的中部位置。如果系统开始变得更加内存紧张，你的进程是有可能被杀死的。

* TRIM_MEMORY_COMPLETE
系统正运行于低内存的状态并且你的进程正处于LRU名单中最容易被杀掉的位置。你应该释放任何不影响你的应用恢复状态的资源。

因为onTrimMemory()的回调是在API 14才被加进来的，对于老的版本，你可以使用onLowMemory)回调来进行兼容。onLowMemory相当与TRIM_MEMORY_COMPLETE。

请注意：当系统开始清除LRU缓存中的进程时，虽然它首先按照LRU的顺序来执行操作，但是它同样会考虑进程的内存使用量以及其他因素。占用越少的进程越容易被留下来。

OnLowMemory()和OnTrimMemory()的比较

1.OnLowMemory被回调时，已经没有后台进程；而onTrimMemory被回调时，还有后台进程。

2.OnLowMemory是在最后一个后台进程被杀时调用，一般情况是low memory killer 杀进程后触发；而OnTrimMemory的触发更频繁，
每次计算进程优先级时，只要满足条件，都会触发。

3.通过一键清理后，OnLowMemory不会被触发，而OnTrimMemory会被触发一次。

使用举例：

       	{% highlight java  %}
@Override
public void onTrimMemory(int level) {
    Log.e(TAG, " onTrimMemory ... level:" + level);     
}

@Override
public void onLowMemory() {     
    Log.e(TAG, " onLowMemory ... ");     
}
          {% endhighlight %}
          
  