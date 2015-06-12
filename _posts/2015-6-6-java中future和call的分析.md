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


下面是一个终极版演示例子。
  {% highlight java %}
  public class CallableAndFuture {     
    
    /** *//**   
     * 自定义一个任务类，实现Callable接口   
     */    
    public static class MyCallableClass implements Callable{     
        // 标志位     
        private int flag = 0;     
        public MyCallableClass(int flag){     
            this.flag = flag;     
        }     
        public String call() throws Exception{     
            if (this.flag == 0){     
                // 如果flag的值为0，则立即返回     
                return "flag = 0";     
            }      
            if (this.flag == 1){     
                // 如果flag的值为1，做一个无限循环     
                try {     
                    while (true) {     
                        System.out.println("looping.");     
                        Thread.sleep(2000);     
                    }     
                } catch (InterruptedException e) {     
                    System.out.println("Interrupted");     
                }     
                return "false";     
            } else {     
                // falg不为0或者1，则抛出异常     
                throw new Exception("Bad flag value!");     
            }     
        }     
    }     
         
    public static void main(String[] args) {     
        // 定义3个Callable类型的任务     
        MyCallableClass task1 = new MyCallableClass(0);     
        MyCallableClass task2 = new MyCallableClass(1);     
        MyCallableClass task3 = new MyCallableClass(2);     
             
        // 创建一个执行任务的服务     
        ExecutorService es = Executors.newFixedThreadPool(3);     
        try {     
            // 提交并执行任务，任务启动时返回了一个 Future对象，     
            // 如果想得到任务执行的结果或者是异常可对这个Future对象进行操作     
            Future future1 = es.submit(task1);     
            // 获得第一个任务的结果，如果调用get方法，当前线程会等待任务执行完毕后才往下执行     
            System.out.println("task1: " + future1.get());     
                 
            Future future2 = es.submit(task2);     
            // 等待5秒后，再停止第二个任务。因为第二个任务进行的是无限循环     
            Thread.sleep(5000);     
            System.out.println("task2 cancel: " + future2.cancel(true));     
                 
            // 获取第三个任务的输出，因为执行第三个任务会引起异常     
            // 所以下面的语句将引起异常的抛出     
            Future future3 = es.submit(task3);     
            System.out.println("task3: " + future3.get());     
        } catch (Exception e){     
            System.out.println(e.toString());     
        }     
        // 停止任务执行服务     
        es.shutdownNow();     
    }     
} 
    {% endhighlight  %}
    
    
    输出结果如下：
    {% highlight c %}
    task1: flag = 0
looping.
looping.
looping.
task2 cancel: true
Interrupted
java.util.concurrent.ExecutionException: java.lang.Exception: Bad flag value!

        {% endhighlight  %}
        
        第一个线程进入call()方法，通过get取得 call返回值和之前的例子没有什么不同。
        第二个线程没有调用get(),而是调用了cancel(true)。结果并没有得到call的返回值，而是cancel覆写后的返回值，同时call()抛出异常。
        第三个线程同样抛出异常。
        
