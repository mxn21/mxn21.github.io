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

### onTerminate的使用

onTerminate()会在app关闭的时候调用,但是就像onDestroy()一样，不能保证一定会被调用。所以最好不要依赖这个方法做重要的处理，
这个方法最多可以用来销毁一写对象，清除一下缓存，但是也并不能保证一定会清除掉，其他操作，例如想在程序结束保存数据，用这个方法明显是错误的。

使用方法如下：

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