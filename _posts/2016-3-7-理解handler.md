---
layout: post
author: mxn
titile: 理解handler
category: 技术博文
tag: android
---

android中处理一些费时的操作需要单独启动一个子线程去处理。子线程处理完毕将结果通知给UI主线程更新UI界面。
子线程与UI主线程的通信在android中使用了消息机制来完成，那么是怎么完成的呢？这就和handler机制的原理（android中跨线程操作的原理，而不是跨进程），
简而言之，就是需要两样古老的东西，消息队列、轮询。也就是说，主线程起来以后有一个消息队列，同时和该队列配对的有一个轮询，
而子线程有这个消息队列的引用，那这样，子线程处理完以后就会向主线程的消息队列发消息，主线程轮询自己的队列，发现有未处理的消息就进行处理。
这就是handler的机制。


### 源码解析

handler其构造函数：

    {% highlight java %}    
    /**
     * Default constructor associates this handler with the {@link Looper} for the
     * current thread.
     *
     * If this thread does not have a looper, this handler won't be able to receive messages
     * so an exception is thrown.
     */
    public Handler() {
        this(null, false);
    }
    /**
     * Constructor associates this handler with the {@link Looper} for the
     * current thread and takes a callback interface in which you can handle
     * messages.
     *
     * If this thread does not have a looper, this handler won't be able to receive messages
     * so an exception is thrown.
     *
     * @param callback The callback interface in which to handle messages, or null.
     */
    public Handler(Callback callback) {
        this(callback, false);
    }
    /**
     * Use the provided {@link Looper} instead of the default one.
     *
     * @param looper The looper, must not be null.
     */
    public Handler(Looper looper) {
        this(looper, null, false);
    }
    /**
     * Use the provided {@link Looper} instead of the default one and take a callback
     * interface in which to handle messages.
     *
     * @param looper The looper, must not be null.
     * @param callback The callback interface in which to handle messages, or null.
     */
    public Handler(Looper looper, Callback callback) {
        this(looper, callback, false);
    }
    /**
     * Use the {@link Looper} for the current thread
     * and set whether the handler should be asynchronous.
     *
     * Handlers are synchronous by default unless this constructor is used to make
     * one that is strictly asynchronous.
     *
     * Asynchronous messages represent interrupts or events that do not require global ordering
     * with respect to synchronous messages.  Asynchronous messages are not subject to
     * the synchronization barriers introduced by {@link MessageQueue#enqueueSyncBarrier(long)}.
     *
     * @param async If true, the handler calls {@link Message#setAsynchronous(boolean)} for
     * each {@link Message} that is sent to it or {@link Runnable} that is posted to it.
     *
     * @hide
     */
    public Handler(boolean async) {
        this(null, async);
    }
    /**
     * Use the {@link Looper} for the current thread with the specified callback interface
     * and set whether the handler should be asynchronous.
     *
     * Handlers are synchronous by default unless this constructor is used to make
     * one that is strictly asynchronous.
     *
     * Asynchronous messages represent interrupts or events that do not require global ordering
     * with respect to synchronous messages.  Asynchronous messages are not subject to
     * the synchronization barriers introduced by {@link MessageQueue#enqueueSyncBarrier(long)}.
     *
     * @param callback The callback interface in which to handle messages, or null.
     * @param async If true, the handler calls {@link Message#setAsynchronous(boolean)} for
     * each {@link Message} that is sent to it or {@link Runnable} that is posted to it.
     *
     * @hide
     */
    public Handler(Callback callback, boolean async) {
        if (FIND_POTENTIAL_LEAKS) {
            final Class<? extends Handler> klass = getClass();
            if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                    (klass.getModifiers() & Modifier.STATIC) == 0) {
                Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                    klass.getCanonicalName());
            }
        }
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
    /**
     * Use the provided {@link Looper} instead of the default one and take a callback
     * interface in which to handle messages.  Also set whether the handler
     * should be asynchronous.
     *
     * Handlers are synchronous by default unless this constructor is used to make
     * one that is strictly asynchronous.
     *
     * Asynchronous messages represent interrupts or events that do not require global ordering
     * with respect to synchronous messages.  Asynchronous messages are not subject to
     * the synchronization barriers introduced by {@link MessageQueue#enqueueSyncBarrier(long)}.
     *
     * @param looper The looper, must not be null.
     * @param callback The callback interface in which to handle messages, or null.
     * @param async If true, the handler calls {@link Message#setAsynchronous(boolean)} for
     * each {@link Message} that is sent to it or {@link Runnable} that is posted to it.
     *
     * @hide
     */
    public Handler(Looper looper, Callback callback, boolean async) {
        mLooper = looper;
        mQueue = looper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
    {% endhighlight %} 
    
构造方法中无参构造和没有传递looper的构造最终调用了public Handler(Callback callback, boolean async)；
在这个方法中获得当前线程的looper，再使用这个looper获得looper上的message queue。

而如果构造中传递了looper，则调用 public Handler(Looper looper, Callback callback, boolean async)，
用传递进来的looper代替默认的looper，然后再同样的初始化mQueue，mCallback，mAsynchronous。

先看mLooper = Looper.myLooper();这一句发生了什么：

    {% highlight java %}   
public static Looper myLooper() {
    return sThreadLocal.get();
}
    {% endhighlight %} 
  
可以看到，该方法返回一个sThreadLocal对象中保存的Looper。如果尚未在当前线程上运行过Looper.prepare()的话，myLooper会返回null。
接下来看看Looper.prepare()的实现：

    {% highlight java %}   
/** Initialize the current thread as a looper.
      * This gives you a chance to create handlers that then reference
      * this looper, before actually starting the loop. Be sure to call
      * {@link #loop()} after calling this method, and end it by calling
      * {@link #quit()}.
      */
    public static void prepare() {
        prepare(true);
    }
    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
     {% endhighlight %} 
     
可以看到该方法只是简单地新建了一个Looper对象，并将其保存在sThreadLocal中。在注释中说明了prepare()要在loop()之前调用，最后
调用quit()。接下来看一下Looper的构造函数。

    {% highlight java %}  
    private Looper(boolean quitAllowed) {
        mQueue = new MessageQueue(quitAllowed);
        mThread = Thread.currentThread();
    }
     {% endhighlight %} 
     
调用完Looper.prepare()后，需调用Looper.loop()才能使消息循环运作起来，其源码如下所示：

    {% highlight java %}  
    /**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;
        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();
        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }
            // This must be in a local variable, in case a UI event sets the logger
            Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }
            msg.target.dispatchMessage(msg);
            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }
            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }
            msg.recycleUnchecked();
        }
    }
         {% endhighlight %} 
         
在loop()主要干了四件事：

1.调用Looper me = myLooper()取出looper对象
2.调用MessageQueue queue = me.mQueue; 取出looper绑定的message queue
3.死循环调用Message msg = queue.next();直到msg为null ，在message queue中取数据
4.在上面的循环中调用msg.target.dispatchMessage(msg); 分发message到指定的target handler

    {% highlight java %}  
  /**
     * Handle system messages here.
     */
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
    {% endhighlight %} 
    
分发之后就执行了mCallback中的回调，这个回调就是我们自己覆写的方法public void handleMessage(Message msg)