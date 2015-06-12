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
 
 那么FutureTask是什么呢
  {% highlight java %}
 public class FutureTask<V> implements RunnableFuture<V> {}
  {% endhighlight  %}  
  可以看到FutureTask继承了RunnableFuture。
  继续看RunnableFuture。
  
   {% highlight java %}
   public interface RunnableFuture<V> extends Runnable, Future<V> {
    /**
     * Sets this Future to the result of its computation
     * unless it has been cancelled.
     */
    void run();
}
   {% endhighlight  %}  
   
   RunnableFuture继承了Runnable和Future。那么也就可以说FutureTask是Future接口的一个具体的实例化。
   FutureTask中覆写了Future中的get(),cancel()等方法。
   
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

   {% endhighlight  %}
   
   可以看到myFutureTast.get()方法得到了Callable的call()方法的返回值，这里是100.
   
   那么get()方法是怎么取得这个call()方法返回的值呢？
   
   继续看看FutureTask的源码。
   首先是FutureTask通过传入Callable实例化，
   
    {% highlight java %}
   
    public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       // ensure visibility of callable
    }
    {% endhighlight  %}
    
    可以看到，传入对象为空的时候抛出异常，不为空的时候将callable保存到自己的变量中，
    因为FutureTask继承了Runnable，所以当new Thread(myFutureTast).start(); 的时候一定会调用FutureTask覆写的run()方法。
    那么找找这个方法。
    
    
     {% highlight java %}
    
    public void run() {
        if (state != NEW ||
            !UNSAFE.compareAndSwapObject(this, runnerOffset,
                                         null, Thread.currentThread()))
            return;
        try {
            Callable<V> c = callable;
            if (c != null && state == NEW) {
                V result;
                boolean ran;
                try {
                    result = c.call();
                    ran = true;
                } catch (Throwable ex) {
                    result = null;
                    ran = false;
                    setException(ex);
                }
                if (ran)
                    set(result);
            }
        } finally {
            // runner must be non-null until state is settled to
            // prevent concurrent calls to run()
            runner = null;
            // state must be re-read after nulling runner to prevent
            // leaked interrupts
            int s = state;
            if (s >= INTERRUPTING)
                handlePossibleCancellationInterrupt(s);
        }
    }
    
    {% endhighlight  %}
    
    可以看到，把之前保存的callable变量，赋值给变量c,然后c调用了call(),把结果给了result ,再调用set(result) ; 
    看看set方法 。 
       {% highlight java %}
    protected void set(V v) {
        if (UNSAFE.compareAndSwapInt(this, stateOffset, NEW, COMPLETING)) {
            outcome = v;
            UNSAFE.putOrderedInt(this, stateOffset, NORMAL); // final state
            finishCompletion();
        }
    }
     {% endhighlight  %}
     
    set()方法把这个result保存到一个新的变量outcome中，outcome就是计算的结果。
    
    那么get()一定是返回了这个outcome 。 
     {% highlight java %}
     public V get() throws InterruptedException, ExecutionException {
        int s = state;
        if (s <= COMPLETING)
            s = awaitDone(false, 0L);
        return report(s);
    }

        {% endhighlight  %}
        
        看到了get()并没有直接返回outcome，而是调用report。
        
        
         {% highlight java %}
          @SuppressWarnings("unchecked")
    private V report(int s) throws ExecutionException {
        Object x = outcome;
        if (s == NORMAL)
            return (V)x;
        if (s >= CANCELLED)
            throw new CancellationException();
        throw new ExecutionException((Throwable)x);
    }
         
           {% endhighlight  %}

在report()中终于返回了之前的结果outcome 。 

以上是对future,call,futureTast源码的走读，可以帮助理解三者之间的逻辑关系。


再回顾最开始的代码
	FutureTask<Integer> myFutureTast = new FutureTask(myCallable); 
	new Thread(myFutureTast).start(); 
	这两行还有另外一种写法，通过线程池来实现。
	
    {% highlight java %}	
public class FutureTest {

	public static void main(String[] args) {
		MyCallable myCallable = new MyCallable() ; 
		//方式1
//		FutureTask<Integer> myFutureTast = new FutureTask(myCallable); 
//		new Thread(myFutureTast).start(); 
		//方式2
		ExecutorService threadPool = Executors.newSingleThreadExecutor();  
		Future<Integer> myFutureTast = threadPool.submit(myCallable) ; 
		
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

运行结果和之前相同

    {% highlight c %}
    已经提交了任务
正在处理任务－－－－>pool-1-thread-1
100
任务执行完毕
     {% endhighlight  %}
