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
就从EvnetBus的入口开始看吧：EventBus.register

    {% highlight java  %}
   public void register(Object subscriber) {
        register(subscriber, false, 0);
    }

     {% endhighlight %}

其实调用的就是同名函数register，它的三个参数意义分别是：
subscriber：就是要注册的一个订阅者，
sticky:表示是否是粘性的，一般默认都是false，除非你调用registerSticky方法了
priority：表示事件的优先级，默认就行，
接下来我们就看看这个函数具体干了什么

    {% highlight java  %}
private synchronized void register(Object subscriber, boolean sticky, int priority) {
        List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriber.getClass());
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod, sticky, priority);
        }
    }
       {% endhighlight %}

<!-- more -->

通过一个findSubscriberMethods方法找到了一个订阅者中的所有订阅方法，返回一个 List<SubscriberMethod>，进入到findSubscriberMethods看看如何实现的

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

对每一个订阅方法，对其调用subscribe方法，进入该方法看看到底干了什么

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

所以对于任何一个订阅者，我们可以找到它的 订阅事件类型列表，通过这个订阅事件类型，可以找到在订阅者中的订阅函数。

register分析完了就分析一下post吧

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

