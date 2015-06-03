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

