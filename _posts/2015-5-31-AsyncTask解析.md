---
layout: post
author: mxn
titile: Async Task解析
category: 技术博文
tag: android
---

Android UI是线程不安全的，如果想要在子线程里进行UI操作，就需要借助Android的异步消息处理机制。不过为了更加方便我们在子线程中更新UI元素，Android从1.5版本就引入了一个AsyncTask类，使用它就可以非常灵活方便地从子线程切换到UI线程。

## 分析AsyncTask的源码

在启动某一个任务之前，要先new出它的实例，因此，先来看一看AsyncTask构造函数中的源码，如下所示：

{% highlight java %}
 public AsyncTask() {
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);
                Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                //noinspection unchecked
                return postResult(doInBackground(mParams));
            }
        };
        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occured while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }
     {% endhighlight  %}   
     
     
只是初始化了两个变量，mWorker和mFuture，并在初始化mFuture的时候将mWorker作为参数传入。mWorker是一个Callable对象，mFuture是一个FutureTask对象，这两个变量会暂时保存在内存中，稍后才会用到它们。
在mWorker的call方法中设置了 mTaskInvoked.set(true);
  从private final AtomicBoolean mTaskInvoked = new AtomicBoolean();可以看出mTaskInvoked是boolean的原子化操作。
  Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);设置了线程优先级。
  doInBackground(mParams)返回的是Result对象，Result传递到postResult中
  {% highlight java %}
   protected abstract Result doInBackground(Params... params);
    {% endhighlight  %}  
    
 <!-- more -->
再来看看postResult。


     {% highlight java %}
    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
        {% endhighlight  %} 

这个方法接受了result之后再将自己本身返回，但是中间实例化了AsyncTaskResult，并且通过message发送出去。

AsyncTaskResult是一个静态内部类：

  {% highlight java %}
  private static class AsyncTaskResult<Data> {
        final AsyncTask mTask;
        final Data[] mData;
        AsyncTaskResult(AsyncTask task, Data... data) {
            mTask = task;
            mData = data;
        }
    }
{% endhighlight  %} 

再看看mFuture，只有一句话，调用FutureTask的get(),传递到postResultIfNotInvoked()
 postResultIfNotInvoked(get());
 
   {% highlight java %}
private void postResultIfNotInvoked(Result result) {
        final boolean wasTaskInvoked = mTaskInvoked.get();
        if (!wasTaskInvoked) {
            postResult(result);
        }
    }
{% endhighlight  %} 
postResultIfNotInvoked中判断是否被调用，如果没有被调用，则执行postResult，发送message出去。

接着如果想要启动某一个任务，就需要调用该任务的execute()方法，因此现在我们来看一看execute()方法的源码，如下所示：
{% highlight java %}
public final AsyncTask<Params, Progress, Result> execute(Params... params) {  
    return executeOnExecutor(sDefaultExecutor, params);  
}  
{% endhighlight  %} 
sDefaultExecutor是一个默认线程池。
 通过 public static final Executor SERIAL_EXECUTOR = new SerialExecutor();
 private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;初始化。
 设置时调用
 {% highlight java %}
 public static void setDefaultExecutor(Executor exec) {
        sDefaultExecutor = exec;
    }
 {% endhighlight  %} 

关于SerialExecutor后面会提到。


execute仅是调用了executeOnExecutor()方法，那么具体的逻辑就应该写在这个方法里了

{% highlight java %}
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,  
        Params... params) {  
    if (mStatus != Status.PENDING) {  
        switch (mStatus) {  
            case RUNNING:  
                throw new IllegalStateException("Cannot execute task:"  
                        + " the task is already running.");  
            case FINISHED:  
                throw new IllegalStateException("Cannot execute task:"  
                        + " the task has already been executed "  
                        + "(a task can be executed only once)");  
        }  
    }  
    mStatus = Status.RUNNING;  
    onPreExecute();  
    mWorker.mParams = params;  
    exec.execute(mFuture);  
    return this;  
}  
{% endhighlight %}

在方法中首先判断了mStatus。在看看mStatus.
  初始化的时候配置了
  private volatile Status mStatus = Status.PENDING;给mStatus赋值。
  

  Status类型是一个枚举类
  {% highlight java %}
  public enum Status {
        /**
         * Indicates that the task has not been executed yet.
         */
        PENDING,
        /**
         * Indicates that the task is running.
         */
        RUNNING,
        /**
         * Indicates that {@link AsyncTask#onPostExecute} has finished.
         */
        FINISHED,
    }
{% endhighlight %}

可以看到status把一个任务分为三种状态：未开始执行，正在执行，已经完成的任务。


观察executeOnExecutor方法，第一次执行之mStatus的值是PENDING。要执行的时候把它设置成RUNNING。如果第二次执行,则会进入 
if (mStatus != Status.PENDING)，然后进入 switch语句，无论哪个 case都抛出异常“Cannot execute task”，这也就是为什么execute（）只能被调用一次。

随后传递 mWorker.mParams = params; 
执行   exec.execute(mFuture); 将前面初始化的mFuture对象传了进去。
可以看到这个执行的execute是SerialExecutor中的execute,下面看看SerialExecutor。
  {% highlight java %}
   private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;
        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }
        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
{% endhighlight %}

可以看到在activity执行excute()时就会执行到SerialExecutor中的execute，注意这个方法有一个Runnable参数，这个参数的值就是mFuture对象。这个方法里的所有逻辑就是在子线程中执行的。
方法中首先创建一个队列类型的mTasks ：final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
然后把mFuture参数再封装到一个Runnable对象当中，然后在把Runnable对象添加到队列当中。mTasks.offer(new Runnable() {}.
之后再调用scheduleNext（）方法。scheduleNext之中先从队列中取出一个任务，如果可以取出不为空的任务，则添加到真正的线程池THREAD_POOL_EXECUTOR之中执行。
那么看看THREAD_POOL_EXECUTOR。

public static final Executor THREAD_POOL_EXECUTOR
            = new ThreadPoolExecutor(CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE,
                    TimeUnit.SECONDS, sPoolWorkQueue, sThreadFactory);
                    
 线程池初始化了一些参数
 
  private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    private static final int CORE_POOL_SIZE = CPU_COUNT + 1;
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    private static final int KEEP_ALIVE = 1;

最大线程数和cpu数量有关，也就是双核手机最多有5个线程。
那么真正执行异步操作在哪里呢。答案就是最开始初始化时候WorkerRunnable对象里面的postResult(doInBackground(mParams));
之前已经观察过，这个方法里面封装了MESSAGE_POST_RESULT的message，然后交给handle
Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
 
                {% highlight java %}  
     private static class InternalHandler extends Handler {
        public InternalHandler() {
            super(Looper.getMainLooper());
        }
        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }
    {% endhighlight %}
    
 handle收到这个类型的message后执行了finish（）
 
        {% highlight java %} 
         private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }
       {% endhighlight %}
 
 在finish（）之中执行onPostExecute（）或者onCancelled（），然后设置mStatus为FINISHED完成状态，因为finish（）是在handler中执行，所以onPostExecute（）也是在主线程中执行。
 
 我们注意到，在刚才InternalHandler的handleMessage()方法里，还有一种MESSAGE_POST_PROGRESS的消息类型，这种消息是用于当前进度的，调用的正是onProgressUpdate()方法。
 
               {% highlight java %} 
       protected final void publishProgress(Progress... values) {
        if (!isCancelled()) {
            getHandler().obtainMessage(MESSAGE_POST_PROGRESS,
                    new AsyncTaskResult<Progress>(this, values)).sendToTarget();
        }
    }
     {% endhighlight %}
     
在doInBackground()方法中调用publishProgress()方法才可以从子线程切换到UI线程，从而完成对UI元素的更新操作。
其实AsyncTask是对handler＋Runnable + Executor做了一个非常好的封装 。 
