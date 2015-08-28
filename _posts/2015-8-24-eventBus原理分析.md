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