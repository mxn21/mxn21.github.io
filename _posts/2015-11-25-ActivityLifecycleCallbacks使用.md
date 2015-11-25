---
layout: post
author: mxn
titile: ActivityLifecycleCallbacks使用
category: 技术博文
tag: android
---

Application通过ActivityLifecycleCallbacks使用接口提供了一套回调方法，用于让开发者对Activity的生命周期事件进行集中处理。
ActivityLifecycleCallbacks接口回调可以简化监测Activity的生命周期事件，在一个类中作统一处理。
ActivityLifecycleCallbacks使用要求API 14+ （Android 4.0+）。

### 使用方法

重写Application的onCreate()方法，或在Application的无参构造方法内，调用Application.registerActivityLifecycleCallbacks()方法
，并实现ActivityLifecycleCallbacks接口。

<!-- more -->

    {% highlight java %}


    {% endhighlight %}
