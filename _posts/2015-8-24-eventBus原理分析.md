---
layout: post
author: mxn
titile: eventBus原理分析
category: 技术博文
tag: android
---


EventBus是一个Android端优化的publish/subscribe消息总线，简化了应用程序内各组件间、组件与后台线程间的通信。
比如请求网络，等网络返回时通过Handler或Broadcast通知UI，两个Fragment之间需要通过Listener通信，这些需求都可以通过EventBus实现。

EventBus的使用是很简单，但是如果我们理解EvnetBus的原理，那么我们就能非常轻松的使用EventBus了。
## 类关系图

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img90.jpg)

以上是 EventBus 主要类的关系图，从中我们也可以看出大部分类都与 EventBus 直接关联。上部分主要是订阅者相关信息，中间是 EventBus 类，
下面是发布者发布事件后的调用。具体类的功能请看下面的详细介绍。

## 核心类功能介绍

###  EventBus.java

EventBus 类负责所有对外暴露的 API，其中的 register、post、unregister 函数配合上自定义的 EventType 及事件响应函数即可完成核心功能.
EventBus 默认可通过静态函数 getDefault 获取单例，当然有需要也可以通过 EventBusBuilder 或 构造函数新建一个 EventBus，
每个新建的 EventBus 发布和订阅事件都是相互隔离的，即一个 EventBus 对象中的发布者发布事件，另一个 EventBus 对象中的订阅者不会收到该订阅。

<!-- more -->

#### register

就从EvnetBus的入口开始看吧：EventBus.register

    {% highlight java  %}
   public void register(Object subscriber) {
        register(subscriber, false, 0);
    }

     {% endhighlight %}

其实调用的就是同名函数register，它的三个参数意义分别是：
subscriber：就是要注册的一个订阅者，
sticky:表示是否是粘性的，一般默认都是false，除非你调用registerSticky方法了,
priority：表示事件的优先级，默认就行，
PS：在此之前的版本 EventBus 还允许自定义事件响应函数名称，这版本中此功能已经被去除。

register 函数流程图如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img89.jpg)

接下来我们就看看这个函数具体干了什么

    {% highlight java  %}
private synchronized void register(Object subscriber, boolean sticky, int priority) {
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriber.getClass());
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod, sticky, priority);
        }
    }
       {% endhighlight %}


register 函数中会先根据订阅者类名去subscriberMethodFinder中查找当前订阅者所有事件响应函数，然后循环每一个事件响应函数，依次执行下面的 subscribe 函数.
subscriberMethodFinde通过一个findSubscriberMethods方法找到了一个订阅者中的所有订阅方法，返回一个 List<SubscriberMethod>，进入到findSubscriberMethods看看如何实现的

    {% highlight java  %}

    List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
    //通过订阅者类名创建一个key
            String key = subscriberClass.getName();
            List<SubscriberMethod> subscriberMethods;
            synchronized (methodCache) {
              //判断是否有缓存，有缓存直接返回缓存
                subscriberMethods = methodCache.get(key);
            }
              //第一次进来subscriberMethods肯定是Null
            if (subscriberMethods != null) {
                return subscriberMethods;
            }
            subscriberMethods = new ArrayList<SubscriberMethod>();
            Class<?> clazz = subscriberClass;
            HashSet<String> eventTypesFound = new HashSet<String>();
            StringBuilder methodKeyBuilder = new StringBuilder();
            while (clazz != null) {
                String name = clazz.getName();
                 //过滤掉系统类
                if (name.startsWith("java.") || name.startsWith("javax.") || name.startsWith("android.")) {
                    // Skip system classes, this just degrades performance
                    break;
                }
            //通过反射，获取到订阅者的所有方法
                // Starting with EventBus 2.2 we enforced methods to be public (might change with annotations again)
                Method[] methods = clazz.getDeclaredMethods();
                for (Method method : methods) {
                    String methodName = method.getName();
                     //只找以onEvent开头的方法
                    if (methodName.startsWith(ON_EVENT_METHOD_NAME)) {
                        int modifiers = method.getModifiers();
                        //判断订阅者是否是public的,并且是否有修饰符，看来订阅者只能是public的，并且不能被final，static等修饰
                        if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
                         //获得订阅函数的参数
                            Class<?>[] parameterTypes = method.getParameterTypes();
                             //看了参数的个数只能是1个
                            if (parameterTypes.length == 1) {
                             //获取onEvent后面的部分
                                String modifierString = methodName.substring(ON_EVENT_METHOD_NAME.length());
                                ThreadMode threadMode;
                                if (modifierString.length() == 0) {
                                //订阅函数为onEvnet
                                //记录线程模型为PostThread,意义就是发布事件和接收事件在同一个线程执行，详细可以参考我对于四个订阅函数不同点分析
                                    threadMode = ThreadMode.PostThread;
                                } else if (modifierString.equals("MainThread")) {
                                /对应onEventMainThread
                                    threadMode = ThreadMode.MainThread;
                                } else if (modifierString.equals("BackgroundThread")) {
                                  //对应onEventBackgrondThread
                                    threadMode = ThreadMode.BackgroundThread;
                                } else if (modifierString.equals("Async")) {
                                //对应onEventAsync
                                    threadMode = ThreadMode.Async;
                                } else {
                                    if (skipMethodVerificationForClasses.containsKey(clazz)) {
                                        continue;
                                    } else {
                                        throw new EventBusException("Illegal onEvent method, check for typos: " + method);
                                    }
                                }
                                //获取参数类型，其实就是接收事件的类型
                                Class<?> eventType = parameterTypes[0];
                                methodKeyBuilder.setLength(0);
                                methodKeyBuilder.append(methodName);
                                methodKeyBuilder.append('>').append(eventType.getName());
                                String methodKey = methodKeyBuilder.toString();
                                if (eventTypesFound.add(methodKey)) {
                                    // Only add if not already found in a sub class
                                     //封装一个订阅方法对象，这个对象包含Method对象，threadMode对象，eventType对象
                                    subscriberMethods.add(new SubscriberMethod(method, threadMode, eventType));
                                }
                            }
                        } else if (!skipMethodVerificationForClasses.containsKey(clazz)) {
                            Log.d(EventBus.TAG, "Skipping method (not public, static or abstract): " + clazz + "."
                                    + methodName);
                        }
                    }
                }
                 //看了还会遍历父类的订阅函数
                clazz = clazz.getSuperclass();
            }
            //最后加入缓存，第二次使用直接从缓存拿
            if (subscriberMethods.isEmpty()) {
                throw new EventBusException("Subscriber " + subscriberClass + " has no public methods called "
                        + ON_EVENT_METHOD_NAME);
            } else {
                synchronized (methodCache) {
                    methodCache.put(key, subscriberMethods);
                }
                return subscriberMethods;
            }
        }

       {% endhighlight %}

对于这个方法的讲解都在注释里面了，这里就不在重复叙述了，到了这里我们就找到了一个订阅者的所有的订阅方法。

我们回到register方法：

    {% highlight java  %}
    for (SubscriberMethod subscriberMethod : subscriberMethods) {
           subscribe(subscriber, subscriberMethod, sticky, priority);
       }
    {% endhighlight %}

对每一个订阅方法，对其调用subscribe方法，

subscribe 函数分三步
第一步：通过subscriptionsByEventType得到该事件类型所有订阅者信息队列，根据优先级将当前订阅者信息插入到订阅者队列subscriptionsByEventType中；
第二步：在typesBySubscriber中得到当前订阅者订阅的所有事件队列，将此事件保存到队列typesBySubscriber中，用于后续取消订阅；
第三步：检查这个事件是否是 Sticky 事件，如果是则从stickyEvents事件保存队列中取出该事件类型最后一个事件发送给当前订阅者。

进入该方法看看到底干了什么

    {% highlight java  %}
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod, boolean sticky, int priority) {
        subscribed = true;
        //从订阅方法中拿到订阅事件的类型
        Class<?> eventType = subscriberMethod.eventType;
        //通过订阅事件类型，找到所有的订阅（Subscription）,订阅中包含了订阅者，订阅方法
        CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
        //创建一个新的订阅
        Subscription newSubscription = new Subscription(subscriber, subscriberMethod, priority);
        //将新建的订阅加入到这个事件类型对应的所有订阅列表
        if (subscriptions == null) {
            //如果该事件目前没有订阅列表，那么创建并加入该订阅
            subscriptions = new CopyOnWriteArrayList<Subscription>();
            subscriptionsByEventType.put(eventType, subscriptions);
        } else {
            //如果有订阅列表，检查是否已经加入过
            for (Subscription subscription : subscriptions) {
                if (subscription.equals(newSubscription)) {
                    throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                            + eventType);
                }
            }
        }

        //根据优先级插入订阅
        int size = subscriptions.size();
        for (int i = 0; i <= size; i++) {
            if (i == size || newSubscription.priority > subscriptions.get(i).priority) {
                subscriptions.add(i, newSubscription);
                break;
            }
        }
        //将这个订阅事件加入到订阅者的订阅事件列表中
        List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
        if (subscribedEvents == null) {
            subscribedEvents = new ArrayList<Class<?>>();
            typesBySubscriber.put(subscriber, subscribedEvents);
        }
        subscribedEvents.add(eventType);
        //这个是对粘性事件的，暂时不讨论
        if (sticky) {
            Object stickyEvent;
            synchronized (stickyEvents) {
                stickyEvent = stickyEvents.get(eventType);
            }
            if (stickyEvent != null) {
                postToSubscription(newSubscription, stickyEvent, Looper.getMainLooper() == Looper.myLooper());
            }
        }
    }
      {% endhighlight %}

好了，到这里差不多register方法分析完了，大致流程就是这样的，我们总结一下：

1、找到被注册者中所有的订阅方法。
2、依次遍历订阅方法，找到EventBus中eventType对应的订阅列表，然后根据当前订阅者和订阅方法创建一个新的订阅加入到订阅列表
3、找到EvnetBus中subscriber订阅的事件列表，将eventType加入到这个事件列表。

所以对于任何一个订阅者，我们可以找到它的订阅事件类型列表，通过这个订阅事件类型，可以找到在订阅者中的订阅函数。

register分析完了就分析一下其他方法吧

#### post

post 函数用于发布事件，cancel 函数用于取消某订阅者订阅的所有事件类型、removeStickyEvent 函数用于删除 sticky 事件。

post 函数流程图如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img91.jpg)

    {% highlight java  %}
    public void post(Object event) {
        //这个EventBus中只有一个，差不多是个单例吧，具体不用细究
        PostingThreadState postingState = currentPostingThreadState.get();
        List<Object> eventQueue = postingState.eventQueue;
        //将事件放入队列
        eventQueue.add(event);

        if (postingState.isPosting) {
            return;
        } else {
            postingState.isMainThread = Looper.getMainLooper() == Looper.myLooper();
            postingState.isPosting = true;
            if (postingState.canceled) {
                throw new EventBusException("Internal error. Abort state was not reset");
            }
            try {
                while (!eventQueue.isEmpty()) {
                    //分发事件
                    postSingleEvent(eventQueue.remove(0), postingState);
                }
            } finally {
                postingState.isPosting = false;
                postingState.isMainThread = false;
            }
        }
    }
      {% endhighlight %}

post里面没有什么具体逻辑，它的功能主要是调用postSingleEvent完成的，进入到这个函数看看吧

    {% highlight java  %}
private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
       Class<? extends Object> eventClass = event.getClass();
    //找到eventClass对应的事件，包含父类对应的事件和接口对应的事件
       List<Class<?>> eventTypes = findEventTypes(eventClass);
       boolean subscriptionFound = false;
       int countTypes = eventTypes.size();
       for (int h = 0; h < countTypes; h++) {
           Class<?> clazz = eventTypes.get(h);
           CopyOnWriteArrayList<Subscription> subscriptions;
           synchronized (this) {
            //找到订阅事件对应的订阅，这个是通过register加入的（还记得吗....）
               subscriptions = subscriptionsByEventType.get(clazz);
           }
           if (subscriptions != null && !subscriptions.isEmpty()) {
               for (Subscription subscription : subscriptions) {
                   postingState.event = event;
                   postingState.subscription = subscription;
                   boolean aborted = false;
                   try {
                    //对每个订阅调用该方法
                       postToSubscription(subscription, event, postingState.isMainThread);
                       aborted = postingState.canceled;
                   } finally {
                       postingState.event = null;
                       postingState.subscription = null;
                       postingState.canceled = false;
                   }
                   if (aborted) {
                       break;
                   }
               }
               subscriptionFound = true;
           }
       }
    //如果没有订阅发现，那么会Post一个NoSubscriberEvent事件
       if (!subscriptionFound) {
           Log.d(TAG, "No subscribers registered for event " + eventClass);
           if (eventClass != NoSubscriberEvent.class && eventClass != SubscriberExceptionEvent.class) {
               post(new NoSubscriberEvent(this, event));
           }
       }
   }
     {% endhighlight %}

这个方法有个核心方法 postToSubscription方法，进入看看吧

    {% highlight java  %}
private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
        //第一个参数就是传入的订阅，第二个参数就是对于的分发事件，第三个参数非常关键：是否在主线程
        switch (subscription.subscriberMethod.threadMode) {
        //这个threadMode是怎么传入的，仔细想想？是不是根据onEvent,onEventMainThread,onEventBackground,onEventAsync决定的？
        case PostThread:
            //直接在本线程中调用订阅函数
            invokeSubscriber(subscription, event);
            break;
        case MainThread:
            if (isMainThread) {
                //如果直接在主线程，那么直接在本现场中调用订阅函数
                invokeSubscriber(subscription, event);
            } else {
                //如果不在主线程，那么通过handler实现在主线程中执行，具体我就不跟踪了
                mainThreadPoster.enqueue(subscription, event);
            }
            break;
        case BackgroundThread:
            if (isMainThread) {
                //如果主线程，创建一个runnable丢入线程池中
                backgroundPoster.enqueue(subscription, event);
            } else {
                //如果子线程，则直接调用
                invokeSubscriber(subscription, event);
            }
            break;
        case Async:
            //不论什么线程，直接丢入线程池
            asyncPoster.enqueue(subscription, event);
            break;
        default:
            throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
        }
    }
       {% endhighlight %}

可以看出：post 函数会首先得到当前线程的 post 信息PostingThreadState，其中包含事件队列，将当前事件添加到其事件队列中，
然后循环调用postSingleEvent函数发布队列中的每个事件。

postSingleEvent 函数会先去eventTypesCache得到该事件对应类型的的父类及接口类型，没有缓存则查找并插入缓存。循环得到的每个类型和接口，调用 postSingleEventForEventType 函数发布每个事件到每个订阅者。

postSingleEventForEventType 函数在subscriptionsByEventType查找该事件订阅者订阅者队列，调用 postToSubscription 函数向每个订阅者发布事件。

postToSubscription 函数中会判断订阅者的 ThreadMode，从而决定在什么 Mode 下执行事件响应函数。

ThreadMode 共有四类：

1.PostThread：默认的 ThreadMode，表示在执行 Post 操作的线程直接调用订阅者的事件响应方法，不论该线程是否为主线程（UI
线程）。当该线程为主线程时，响应方法中不能有耗时操作，否则有卡主线程的风险。适用场景：对于是否在主线程执行无要求，但若 Post 线程为主线程，
不能耗时的操作；

2.MainThread：在主线程中执行响应方法。如果发布线程就是主线程，则直接调用订阅者的事件响应方法，否则通过主线程的 Handler
发送消息在主线程中处理——调用订阅者的事件响应函数。显然，MainThread类的方法也不能有耗时操作，以避免卡主线程。适用场景：必须在主线程执行的操作；

3.BackgroundThread：在后台线程中执行响应方法。如果发布线程不是主线程，则直接调用订阅者的事件响应函数，否则启动唯一的后台线程去处理。
由于后台线程是唯一的，当事件超过一个的时候，它们会被放在队列中依次执行，因此该类响应方法虽然没有PostThread类和MainThread类方法对性能敏感，
但最好不要有重度耗时的操作或太频繁的轻度耗时操作，以造成其他操作等待。适用场景：操作轻微耗时且不会过于频繁，即一般的耗时操作都可以放在这里；

4.Async：不论发布线程是否为主线程，都使用一个空闲线程来处理。和BackgroundThread不同的是，Async类的所有线程是相互独立的，
因此不会出现卡线程的问题。适用场景：长耗时操作，例如网络访问。

#### 主要成员变量含义

1.defaultInstance默认的 EventBus 实例，根据EventBus.getDefault()函数得到。
2.DEFAULT_BUILDER默认的 EventBus Builder。
3.eventTypesCache事件对应类型及其父类和实现的接口的缓存，以 eventType 为 key，元素为 Object 的 ArrayList 为 Value，Object 对象为 eventType 的父类或接口。
4.subscriptionsByEventType事件订阅者的保存队列，以 eventType 为 key，元素为Subscription的 ArrayList 为 Value，其中Subscription为订阅者信息，由 subscriber, subscriberMethod, priority 构成。
5.typesBySubscriber订阅者订阅的事件的保存队列，以 subscriber 为 key，元素为 eventType 的 ArrayList 为 Value。
6.stickyEventsSticky 事件保存队列，以 eventType 为 key，event 为元素，由此可以看出对于同一个 eventType 最多只会有一个 event 存在。
7.currentPostingThreadState当前线程的 post 信息，包括事件队列、是否正在分发中、是否在主线程、订阅者信息、事件实例、是否取消。
8.mainThreadPoster、backgroundPoster、asyncPoster事件主线程处理者、事件 Background 处理者、事件异步处理者。
9.subscriberMethodFinder订阅者响应函数信息存储和查找类。
10.executorService异步和 BackGround 处理方式的线程池。
11.throwSubscriberException当调用事件处理函数异常时是否抛出异常，默认为 false，建议通过EventBus.builder().throwSubscriberException(true).installDefaultEventBus()打开。
12.logSubscriberExceptions当调用事件处理函数异常时是否打印异常信息，默认为 true。
13.logNoSubscriberMessages当没有订阅者订阅该事件时是否打印日志，默认为 true。
14.sendSubscriberExceptionEvent当调用事件处理函数异常时是否发送 SubscriberExceptionEvent 事件，若此开关打开，
订阅者可通过public void onEvent(SubscriberExceptionEvent event)订阅该事件进行处理，默认为 true。
15.sendNoSubscriberEvent当没有事件处理函数对事件处理时是否发送 NoSubscriberEvent 事件，若此开关打开，
订阅者可通过public void onEvent(NoSubscriberEvent event)订阅该事件进行处理，默认为 true。
16.eventInheritance是否支持事件继承，默认为 true。

### EventBusBuilder.java

跟一般 Builder 类似，用于在需要设置参数过多时构造 EventBus。包含的属性也是 EventBus 的一些设置参数，意义见EventBus.java的介绍，build 函数用于新建
EventBus 对象，installDefaultEventBus 函数将当前设置应用于 Default EventBus。

### SubscriberMethodFinder.java

订阅者响应函数信息存储和查找类，由 HashMap 缓存，以 ${subscriberClassName} 为 key，SubscriberMethod 对象为元素的 ArrayList 为 value。findSubscriberMethods 函数用于查找订阅者响应函数，如果不在缓存中，则遍历自己的每个函数并递归父类查找，查找成功后保存到缓存中。遍历及查找规则为：
a. 遍历 subscriberClass 每个方法；
b. 该方法不以java.、javax.、android.这些 SDK 函数开头，并以onEvent开头，表示可能是事件响应函数继续，否则检查下一个方法；
c. 该方法是否是 public 的，并且不是 ABSTRACT、STATIC、BRIDGE、SYNTHETIC 修饰的，满足条件则继续。其中 BRIDGE、SYNTHETIC 为编译器生成的一些函数修饰符；
d. 该方法是否只有 1 个参数，满足条件则继续；
e. 该方法名为 onEvent 则 threadMode 为ThreadMode.PostThread；
该方法名为 onEventMainThread 则 threadMode 为ThreadMode.MainThread；
该方法名为 onEventBackgroundThread 则 threadMode 为ThreadMode.BackgroundThread；
该方法名为 onEventAsync 则 threadMode 为ThreadMode.Async；
其他情况且不在忽略名单 (skipMethodVerificationForClasses) 中则抛出异常。
f. 得到该方法唯一的参数即事件类型 eventType，将这个方法、threadMode、eventType 一起构造 SubscriberMethod 对象放到 ArrayList 中。
g. 回到 b 遍历 subscriberClass 的下一个方法，若方法遍历结束到 h；
h. 回到 a 遍历自己的父类，若父类遍历结束回到 i；
i. 若 ArrayList 依然为空则抛出异常，否则会将 ArrayList 做为 value，${subscriberClassName} 做为 key 放到缓存 HashMap 中。 对于事件函数的查找有两个小的性能优化点：
第一次查找后保存到了缓存中，即上面介绍的 HashMap;遇到 java. javax. android. 开头的类会自动停止查找.
类中的 skipMethodVerificationForClasses 属性表示跳过哪些类中非法以 onEvent 开头的函数检查，若不跳过则会抛出异常。
PS：在此之前的版本 EventBus 允许自定义事件响应函数名称，缓存的 HashMap key 为 ${subscriberClassName}.${eventMethodName}，这版本中此功能已经被去除。

### SubscriberMethod.java

订阅者事件响应函数信息，包括响应方法、线程 Mode、事件类型以及一个用来比较 SubscriberMethod 是否相等的特征值 methodString 共四个变量，
其中 methodString 为 ${methodClassName}#${methodName}(${eventTypeClassName}。

### Subscription.java

订阅者信息，包括 subscriber 对象、事件响应方法 SubscriberMethod、优先级 priority。

### HandlerPoster.java

事件主线程处理，对应ThreadMode.MainThread。继承自 Handler，enqueue 函数将事件放到队列中，并利用 handler 发送 message，handleMessage 函数从队列中取事件，invoke 事件响应函数处理。


###
###
###
###
###
###


