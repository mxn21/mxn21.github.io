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
    
            String key = subscriberClass.getName();
            List<SubscriberMethod> subscriberMethods;
            synchronized (methodCache) {
                subscriberMethods = methodCache.get(key);
            }
            if (subscriberMethods != null) {
                return subscriberMethods;
            }
            subscriberMethods = new ArrayList<SubscriberMethod>();
            Class<?> clazz = subscriberClass;
            HashSet<String> eventTypesFound = new HashSet<String>();
            StringBuilder methodKeyBuilder = new StringBuilder();
            while (clazz != null) {
                String name = clazz.getName();
                if (name.startsWith("java.") || name.startsWith("javax.") || name.startsWith("android.")) {
                    // Skip system classes, this just degrades performance
                    break;
                }

                // Starting with EventBus 2.2 we enforced methods to be public (might change with annotations again)
                Method[] methods = clazz.getDeclaredMethods();
                for (Method method : methods) {
                    String methodName = method.getName();
                    if (methodName.startsWith(ON_EVENT_METHOD_NAME)) {
                        int modifiers = method.getModifiers();
                        if ((modifiers & Modifier.PUBLIC) != 0 && (modifiers & MODIFIERS_IGNORE) == 0) {
                            Class<?>[] parameterTypes = method.getParameterTypes();
                            if (parameterTypes.length == 1) {
                                String modifierString = methodName.substring(ON_EVENT_METHOD_NAME.length());
                                ThreadMode threadMode;
                                if (modifierString.length() == 0) {
                                    threadMode = ThreadMode.PostThread;
                                } else if (modifierString.equals("MainThread")) {
                                    threadMode = ThreadMode.MainThread;
                                } else if (modifierString.equals("BackgroundThread")) {
                                    threadMode = ThreadMode.BackgroundThread;
                                } else if (modifierString.equals("Async")) {
                                    threadMode = ThreadMode.Async;
                                } else {
                                    if (skipMethodVerificationForClasses.containsKey(clazz)) {
                                        continue;
                                    } else {
                                        throw new EventBusException("Illegal onEvent method, check for typos: " + method);
                                    }
                                }
                                Class<?> eventType = parameterTypes[0];
                                methodKeyBuilder.setLength(0);
                                methodKeyBuilder.append(methodName);
                                methodKeyBuilder.append('>').append(eventType.getName());
                                String methodKey = methodKeyBuilder.toString();
                                if (eventTypesFound.add(methodKey)) {
                                    // Only add if not already found in a sub class
                                    subscriberMethods.add(new SubscriberMethod(method, threadMode, eventType));
                                }
                            }
                        } else if (!skipMethodVerificationForClasses.containsKey(clazz)) {
                            Log.d(EventBus.TAG, "Skipping method (not public, static or abstract): " + clazz + "."
                                    + methodName);
                        }
                    }
                }
                clazz = clazz.getSuperclass();
            }
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