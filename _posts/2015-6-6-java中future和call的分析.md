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
