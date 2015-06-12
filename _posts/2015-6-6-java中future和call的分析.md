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


 Callable接口类似于Runnable，从名字就可以看出来了，但是Runnable不会返回结果，并且无法抛出返回结果的异常。
 而Callable功能更强大一些，被线程执行后，可以返回值，这个返回值可以被Future拿到，也就是说，Future可以拿到异步执行任务的返回值。
 
 
 下面看一个例子
 
 {% highlight java %}
 public class MyCallable implements Callable<Integer>{

	@Override
	public Integer call() throws Exception {
		System.out.println("正在处理任务－－－－>" + Thread.currentThread().getName());
		Thread.sleep(5000) ; 
		return 100;
	}

}
 {% endhighlight  %}  


 {% highlight java %}
 public class FutureTest {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		MyCallable myCallable = new MyCallable() ; 
		
		FutureTask<Integer> myFutureTast = new FutureTask(myCallable); 
		
		new Thread(myFutureTast).start(); 
		
		
		System.out.println("已经提交了任务");
		try {
			System.out.println(myFutureTast.get());
		} catch (InterruptedException | ExecutionException e) {
			e.printStackTrace();
		}
		System.out.println("任务执行完毕");
	}

}
 
 
  {% endhighlight  %}  
  输出结果：
   {% highlight c %}
  已经提交了任务
  正在处理任务－－－－>Thread-0
  100
  任务执行完毕

   {% highlight c %}
   
   可以看到myFutureTast.get()方法得到了Callable的call()方法的返回值，这里是100.
