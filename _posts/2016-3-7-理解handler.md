---
layout: post
author: mxn
titile: 理解handler
category: 技术博文
tag: android
---

android中处理一些费时的操作需要单独启动一个子线程去处理。子线程处理完毕将结果通知给UI主线程更新UI界面。
子线程与UI主线程的通信在android中使用了消息机制来完成，那么是怎么完成的呢？这就和handler机制的原理（android中线程间通信的原理，而不是进程间通信），
简而言之，就是需要两样古老的东西，消息队列、轮询。也就是说，主线程起来以后有一个消息队列，同时和该队列配对的有一个轮询，
而子线程有这个消息队列的引用，那这样，子线程处理完以后就会向主线程的消息队列发消息，主线程轮询自己的队列，发现有未处理的消息就进行处理。
这就是handler的机制。

下面介绍Handler，Message，MessageQueue，Looper的关系：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img161.jpg)

looper对象拥有message queue，并且负责从message queue中取出消息给handler来处理。同时handler又负责发送message给looper,由looper把message添加到message queue尾部。
handler和looper是来联系在一起的。多个message可以指向同一个handler，多个handler也可以指向同一个looper。

### Handler

消息的处理者，handler负责将需要传递的信息封装成Message，通过调用handler对象的obtainMessage()来实现。将消息传递给Looper，
这是通过handler对象的sendMessage()来实现的。继而由Looper将Message放入MessageQueue中。当Looper对象看到MessageQueue中含有Message，
就将其分发出去。该handler对象收到该消息后，调用相应的handler对象的handleMessage()方法对其进行处理。

Handler一些特点：handler可以分发Message对象和Runnable对象到主线程中,每个Handler实例,都会绑定到创建他的线程中(一般是位于主线程),
它有两个作用: (1)安排消息或Runnable在某个主线程中某个地方执行。(2)安排一个动作在不同的线程中执行。

### Message

消息对象，Message Queue中的存放的对象。一个Message Queue中包含多个Message。Message实例对象的取得，
通常使用Message类里的静态方法obtain(),该方法有多个重载版本可供选择；它的创建并不一定是直接创建一个新的实例，
而是先从Message Pool(消息池)中看有没有可用的Message实例，存在则直接取出返回这个实例。如果Message Pool中没有可用的Message实例，
则才用给定的参数创建一个Message对象。调用removeMessages()时，将Message从Message Queue中删除，同时放入到Message Pool中。
除了上面这种方式，也可以通过Handler对象的obtainMessage()获取一个Message实例。

### MessageQueue

消息队列，存放消息的地方。每一个线程最多只可以拥有一个MessageQueue数据结构。创建一个线程的时候，并不会自动创建其MessageQueue。
通常使用一个Looper对象对该线程的MessageQueue进行管理。主线程创建时，会创建一个默认的Looper对象，而Looper对象的创建，
将自动创建一个Message Queue。其他非主线程，不会自动创建Looper，要需要的时候，通过调用Looper的prepare函数来实现。 

### Looper

Looper是MessageQueue的管理者。每一个MessageQueue都不能脱离Looper而存在，Looper对象的创建是通过prepare函数来实现的。
同时每一个Looper对象和一个线程关联。通过调用Looper.myLooper()可以获得当前线程的Looper对象 创建一个Looper对象时，
会同时创建一个MessageQueue对象。除了主线程有默认的Looper，其他线程默认是没有MessageQueue对象的，所以，不能接受Message。
如需要接受，自己定义一个Looper对象(通过prepare函数),这样该线程就有了自己的Looper对象和MessageQueue数据结构了,而且一个线程只能有一个looper。
调用完prepare以后，此线程就成为了所谓的LooperThread,若在当前LooperThread中创建Handler对象，那么此Handler会自动关联到当前线程的looper对象，也就是拥有looper的引用。
Looper从MessageQueue中取出Message然后，交由Handler的handleMessage进行处理。处理完成后，调用Message.recycle()将其放入Message Pool中。

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
3.死循环调用Message msg = queue.next();在message queue中取数据，若msg为null就不执行下面的分发，继续死循环
4.在上面的循环中若msg不为null，调用msg.target.dispatchMessage(msg); 分发message到指定的target handler

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
    
分发之后就执行了mCallback中的回调，这个回调就是我们自己覆写的方法public void handleMessage(Message msg)。

从上面代码也可以看出，轮询器在for (;;){}死循环代码块中不断的执行，通过queue.next();从MessageQueue中取出一个Message，
当msg不为空时，执行msg.target.dispatchMessage(msg);（实际上最终是调用到了Handler的dispatchMessage方法去拦截消息）

在Message中的源码中，target就是当前线程的Handler对象，msg的成员变量target是在发送消息的时候设置好的，一般就通过哪个Handler
来发送消息，就通过哪个Handler来处理消息。

接下来看一下Message类：

    {% highlight java %}  
public final class Message implements Parcelable {
    public int what;
    public int arg1;
    public int arg2;
    public Object obj;
    ...
    /*package*/ int flags;
    /*package*/ long when;
    /*package*/ Bundle data;
    /*package*/ Handler target;
    /*package*/ Runnable callback;
    /*package*/ Message next;
    private static final Object sPoolSync = new Object();
    private static Message sPool;
    private static int sPoolSize = 0;
}
    {% endhighlight %} 
    
Message中有一个静态类型的Message对象，叫做sPool，同时还有一个叫做next的Message对象，这个应该是指向队列中下一个message的引用。
当我们调用Message.obtain()时，返回了一个Message对象。Message对象使用完毕后，调用recycle()方法将其回收。其中obtain方法的代码如下所示：
        
            {% highlight java %} 
        /**
         * Return a new Message instance from the global pool. Allows us to
         * avoid allocating new objects in many cases.
         */
        public static Message obtain() {
            synchronized (sPoolSync) {
                if (sPool != null) {
                    Message m = sPool;
                    sPool = m.next;
                    m.next = null;
                    m.flags = 0; // clear in-use flag
                    sPoolSize--;
                    return m;
                }
            }
            return new Message();
        }
            {% endhighlight %} 
            
可以看到，obtain方法被调用时，首先检测sPool对象是否为空，若不为空将其当做新的message对象返回，并指向message对象的next属性，
sPoolSize自减。可以看出message对象通过next属性串成了一个链表，sPool为“头指针”。再来看看recycle方法的实现。

    {% highlight java %} 
    /**
     * Return a Message instance to the global pool.
     * <p>
     * You MUST NOT touch the Message after calling this function because it has
     * effectively been freed.  It is an error to recycle a message that is currently
     * enqueued or that is in the process of being delivered to a Handler.
     * </p>
     */
    public void recycle() {
        if (isInUse()) {
            if (gCheckRecycle) {
                throw new IllegalStateException("This message cannot be recycled because it "
                        + "is still in use.");
            }
            return;
        }
        recycleUnchecked();
    }
    /**
     * Recycles a Message that may be in-use.
     * Used internally by the MessageQueue and Looper when disposing of queued Messages.
     */
    void recycleUnchecked() {
        // Mark the message as in use while it remains in the recycled object pool.
        // Clear out all other details.
        flags = FLAG_IN_USE;
        what = 0;
        arg1 = 0;
        arg2 = 0;
        obj = null;
        replyTo = null;
        sendingUid = -1;
        when = 0;
        target = null;
        callback = null;
        data = null;
        synchronized (sPoolSync) {
            if (sPoolSize < MAX_POOL_SIZE) {
                next = sPool;
                sPool = this;
                sPoolSize++;
            }
        }
    }
    {% endhighlight %} 

如果message对象不是处于正在被使用的状态，则会被回收。其属性全部恢复到原始状态后，放在了链表的头部。sPool对象“指向”它，sPoolSize自增。
可以看出，通过obtain和recycle方法可以重用message对象。通过操作next、sPoolSync、sPool、sPoolSize这四个属性，实现了一个类似栈的对象池。

下面看看msg对象是如何放到消息队列里面的,通常来说，我们通过Handler的sendMessage(msg)方法来发送消息，其源码如下所示：

    {% highlight java %} 
public final boolean sendMessage(Message msg)
{
    return sendMessageDelayed(msg, 0);
}
...
public final boolean sendMessageDelayed(Message msg, long delayMillis)
{
    if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}
...
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}
...
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
    {% endhighlight %} 
    
可知sendMessage最终会调用queue.enqueueMessage(msg, uptimeMillis)将msg对象保存至message queue中，uptimeMillis表示msg执行回调的时刻。 我们来看一下MessageQueue类的enqueueMessage方法：

    {% highlight java %} 
boolean enqueueMessage(Message msg, long when) {
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }
    synchronized (this) {
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w("MessageQueue", e.getMessage(), e);
            msg.recycle();
            return false;
        }
        // 1.设置当前msg的状态
        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        // 2.检测当前头指针是否为空（队列为空）或者没有设置when 或者设置的when比头指针的when要前
        if (p == null || when == 0 || when < p.when) {
            // 3. 插入队列头部，并且唤醒线程处理msg
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            //4. 几种情况要唤醒线程处理消息：1）队列是堵塞的 2)barrier，头部结点无target 3）当前msg是堵塞的
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            // 5. 将当前msg插入第一个比其when值大的结点前。
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }
        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
    {% endhighlight %} 
    
结合注释，我们可以了解到msg push到queue中时，queue的状态的变化和处理队列的逻辑。
前文中Looper对象的loop方法中：

    {% highlight java %} 
for (;;) {
    ...
    Message msg = queue.next(); // 3. 堵塞式在message queue中取数据
    if (msg == null) {
        // No message indicates that the message queue is quitting.
        return;
    }
    ...
    msg.target.dispatchMessage(msg); 4. 分发message到指定的target handler
    ...
}
    {% endhighlight %} 
    
可以看出，message queue的next方法被调用时，可能会发生堵塞。我们来看一看message queue的next方法：

    {% highlight java %} 
Message next() {
    // 1. 判断当前loop是否已经使用过，下文会解释这个mPtr
    final long ptr = mPtr;
    if (ptr == 0) {
        return null;
    }
    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    int nextPollTimeoutMillis = 0;
    // 2. 进入死循环，直到获取到合法的msg对象为止。
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands(); // 这个是什么？
        }
        // 3. 进入等待，nextPollTimeoutMillis为等待超时值
        nativePollOnce(ptr, nextPollTimeoutMillis);
        synchronized (this) {
            // 4. 获取下一个msg
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            if (msg != null && msg.target == null) {
                // 当前节点为barrier，所以要找到第一个asynchronous节点
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
            if (msg != null) {
                if (now < msg.when) {
                    // 当前队列里最早的节点比当前时间还要晚，所以要进入堵塞状态，超时值为nextPollTimeoutMillis
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // 删除当前节点，并返回
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (false) Log.v("MessageQueue", "Returning message: " + msg);
                    return msg;
                }
            } else {
                // 头结点指向null
                nextPollTimeoutMillis = -1;
            }
            // Process the quit message now that all pending messages have been handled.
            if (mQuitting) {
                dispose();
                return null;
            }
            // 5. 如果当前状态为idle（就绪），则进入idle handle的代码块
            //    进入idle的情况有：队列为空；队列头元素blocking;
            if (pendingIdleHandlerCount < 0
                    && (mMessages == null || now < mMessages.when)) {
                pendingIdleHandlerCount = mIdleHandlers.size();
            }
            if (pendingIdleHandlerCount <= 0) {
                // 6. 本轮唤醒（next被调用）时没处理任何东西，故再次进入等待。
                mBlocked = true;
                continue;
            }
            if (mPendingIdleHandlers == null) {
                mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
            }
            mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
        }
        // 在一次next调用中，这个代码块只会执行一次
        for (int i = 0; i < pendingIdleHandlerCount; i++) {
            final IdleHandler idler = mPendingIdleHandlers[i];
            mPendingIdleHandlers[i] = null; // release the reference to the handler
            boolean keep = false;
            try {
                // 如果返回true，则idler被保留，下次next的idle时会被调用。
                keep = idler.queueIdle();
            } catch (Throwable t) {
                Log.wtf("MessageQueue", "IdleHandler threw exception", t);
            }
            if (!keep) {
                synchronized (this) {
                    mIdleHandlers.remove(idler);
                }
            }
        }
        // Reset the idle handler count to 0 so we do not run them again.
        pendingIdleHandlerCount = 0;
        // While calling an idle handler, a new message could have been delivered
        // so go back and look again for a pending message without waiting.
        nextPollTimeoutMillis = 0;
    }
}
    {% endhighlight %} 
    
代码执行流程见注释。其中IdleHandler是一个接口：

    {% highlight java %} 
public static interface IdleHandler {
    boolean queueIdle();
}
    {% endhighlight %} 
    
IdleHandler提供了一个在MessageQueue进入idle时的一个hook point。更多时与barrier机制一起使用，使message queue遇到barrier时产生一个回调。

### 总结

前面涉及到的几个主要的类Handler、Looper、MessageQueue和Message的关系如下所述：

Handler负责将Looper绑定到线程，初始化Looper和提供对外API。
Looper负责消息循环和操作MessageQueue对象。
MessageQueue实现了一个堵塞队列。
Message是一次业务中所有参数的载体。



参考原文：

[Android应用程序消息处理机制](http://legendmohe.net/2015/06/12/android应用程序消息处理机制/)

[Handler源码解析 (Java层)](http://legendmohe.net/2015/10/23/android-handler源码解析-java层/)

[Handler源码解析 (Native层)](http://legendmohe.net/2015/10/26/android-handler源码解析-native层/)