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
       register(subscriber, DEFAULT_METHOD_NAME, false, 0);
   }
     {% endhighlight %}

<!-- more -->