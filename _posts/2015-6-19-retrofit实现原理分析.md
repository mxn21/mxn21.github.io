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

{% highlight java %}
private void ensureSaneDefaults() {
    if (converter == null) {
      converter = Platform.get().defaultConverter();
    }
    if (clientProvider == null) {
      clientProvider = Platform.get().defaultClient();
    }
    if (httpExecutor == null) {
      httpExecutor = Platform.get().defaultHttpExecutor();
    }
    if (callbackExecutor == null) {
      callbackExecutor = Platform.get().defaultCallbackExecutor();
    }
    if (errorHandler == null) {
      errorHandler = ErrorHandler.DEFAULT;
    }
    if (log == null) {
      log = Platform.get().defaultLog();
    }
    if (requestInterceptor == null) {
      requestInterceptor = RequestInterceptor.NONE;
    }
  }
}
{% endhighlight  %}

 <!-- more -->
 
 可以看到进行初始化的时候调用了Platform.get()。
 
 {% highlight java %}
 
 private static final Platform PLATFORM = findPlatform();
 
static Platform get() {
  return PLATFORM;
}
 
private static Platform findPlatform() {
  try {
    Class.forName("android.os.Build");//只要android.os.Build的class可以正常找到,证明是在android平台
    if (Build.VERSION.SDK_INT != 0) {
      return new Android();
    }
  } catch (ClassNotFoundException ignored) {
  }
 
  if (System.getProperty("com.google.appengine.runtime.version") != null) {
    return new AppEngine();   //google的app Engine平台
  }
 
  return new Base();
}　　


 {% endhighlight  %}
 
 由于retrofit支持不同的平台，Platform用于判断使用的是哪个平台， retrofit的Android类继承了Platform, 根据android的特性对配置项做了处理.
  {% highlight java %}
 private static class Android extends Platform {
    @Override Converter defaultConverter() {
      return new GsonConverter(new Gson());  //默认的转换器是Gson
    }
 
    @Override Client.Provider defaultClient() {
      final Client client;
      if (hasOkHttpOnClasspath()) {  //有okhttp的路径就使用 Okhttp
        client = OkClientInstantiator.instantiate();
      } else if (Build.VERSION.SDK_INT < Build.VERSION_CODES.GINGERBREAD) {
        client = new AndroidApacheClient(); //没有okhttp,且版本小于2.3 使用HttpClient
      } else {
        client = new UrlConnectionClient();  //没有okhttp,且版本大于等于2.3 使用urlConnection.
      }
      return new Client.Provider() {
        @Override public Client get() {
          return client;
        }
      };
    }
 
    @Override Executor defaultHttpExecutor() {   //网络访问执行的线程.
      return Executors.newCachedThreadPool(new ThreadFactory() {  //一个cached的线程池.可以复用老线程且线程长时间不用会自动回收. 线程池中线程不够会生成新线程.
        @Override public Thread newThread(final Runnable r) {
          return new Thread(new Runnable() {
            @Override public void run() {
              Process.setThreadPriority(THREAD_PRIORITY_BACKGROUND);    //设置线程的优先级 为最低
              r.run();
            }
          }, RestAdapter.IDLE_THREAD_NAME);
        }
      });
    }
 
    @Override Executor defaultCallbackExecutor() { //异步执行的线程.
      return new MainThreadExecutor();
    }
 
    @Override RestAdapter.Log defaultLog() {  //通过Log.d("Retrofit",String)打印log
      return new AndroidLog("Retrofit");
    }
  }
  
   {% endhighlight  %}
   
   其中判断了是否有okHttp的路径
     {% highlight java %}
   private static boolean hasOkHttpOnClasspath() {
  try {
    Class.forName("com.squareup.okhttp.OkHttpClient"); //是否可以找到OkHttpClient类.
    return true;
  } catch (ClassNotFoundException ignored) {
  }
  return false;
}

   {% endhighlight  %}
