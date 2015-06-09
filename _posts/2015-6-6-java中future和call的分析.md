---
layout: post
author: mxn
titile: 理解java中的future,futureTask和callable
category: 技术博文
tag: android
---

在Android的AsyncTask源码中用到了future,futureTask和callable。要理解源码，首先要理解这三个类的关系和用法。

先看看future
{% highlight java %}

public interface Future<V> {
  boolean cancel(boolean mayInterruptIfRunning);
  boolean isCancelled();
  boolean isDone();
 V get() throws InterruptedException, ExecutionException;
 V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}

     {% endhighlight  %}   

可以看到future是一个接口，里面定义了一些方法。

再看看callable
{% highlight java %}
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}

{% endhighlight  %}  

他也是一个接口，里面定义了一个call方法。


 Callable接口类似于Runnable，从名字就可以看出来了，但是Runnable不会返回结果，并且无法抛出返回结果的异常
