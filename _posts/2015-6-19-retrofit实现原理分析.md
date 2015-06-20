---
layout: post
author: mxn
titile: retrofit实现原理分析
category: 技术博文
tag: java
---

retrofit有几个关键的地方.

1.用户自定义的接口和接口方法.(由动态代理创建对象.)

2.converter转换器.(把response转换为一个具体的对象)

3.注解的使用.

让我们跟随Api来看吧.

RestAdapter restAdapter = new RestAdapter.Builder().setEndpoint(API_URL).build(); 

build()其内部实现是这样的:
{% highlight java %}
public RestAdapter build() {
     if (endpoint == null) {
       throw new IllegalArgumentException("Endpoint may not be null.");
     }
     ensureSaneDefaults();
     return new RestAdapter(endpoint, clientProvider, httpExecutor, callbackExecutor,
         requestInterceptor, converter, profiler, errorHandler, log, logLevel);
   }
   
{% endhighlight  %}

当用户没有设置自定义的converter,client, httpExecutor(http访问执行的线程-->只对异步的retrofit有效.), callBackExecutor(异步的callBack执行的线程), errorHandler, log, RequestInterceptor的时候,就会使用retrofit默认的配置.调用    ensureSaneDefaults();
