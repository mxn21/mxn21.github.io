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
   
   可以发现上面默认的Http的Executor是一个线程池.

而CallBack的Executor是在主线程执行的. 由绑定MainLooper的Handler提交到主线程执行.
    {% highlight java %}
    
    public final class MainThreadExecutor implements Executor {
  private final Handler handler = new Handler(Looper.getMainLooper()); //关联主线程的Handler
 
  @Override public void execute(Runnable r) {
    handler.post(r);                                 //提交到主线程执行
  }
}
   {% endhighlight  %}
   
   
   {% highlight java %}

public final boolean post(Runnable r)
    {
       return  sendMessageDelayed(getPostMessage(r), 0);
    }
    {% endhighlight  %}
   
   {% highlight java %}
   
   private static Message getPostMessage(Runnable r) {   //把runnable封装到Message中.
      Message m = Message.obtain();
      m.callback = r;
      return m;
  }
    {% endhighlight  %}
    
    现在有下面一个接口,
      {% highlight java %}
    interface SimplePOST{
      @POST("/android") 
      Response getResponse();
  }　
  
   {% endhighlight  %}
  
  
    下面了解下 SimplePOST simplePost= adapter.create(SimplePOST.class)的内部逻辑.
       {% highlight java %}
    public <T> T create(Class<T> service) {
    Utils.validateServiceClass(service);                                                      
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },   //动态代理
        new RestHandler(getMethodInfoCache(service)));
  }
      {% endhighlight  %}
      
          {% highlight java %}
      static <T> void validateServiceClass(Class<T> service) {      //确保create()参数是一个接口类型,且这个接口没有继承其他的接口.(使用动态代理的前提)
   if (!service.isInterface()) {
     throw new IllegalArgumentException("Only interface endpoint definitions are supported.");
   }
   if (service.getInterfaces().length > 0) {
     throw new IllegalArgumentException("Interface definitions must not extend other interfaces.");
   }
 }
      {% endhighlight  %}
    
　可以发现adapter.create()方法的内部实现是利用动态代理生成了service接口的一个实现类. 根据动态代理的原理. 可以得知调用实现类的方法其实就是调用InvocationtHandler的对应方法.
　
　　虽然这里是运用了动态代理的技术.但是却和一般的动态代理不一样. 一般的动态代理的InvocationHandler应该通过构造函数中传入委托类A.然后在invoke方法中调用A的方法, 但这里是没有委托类的.只是利用动态代理自动生成接口的实现类.

　　因为java的动态代理是基于接口的,所以retrofit也要求用户自定义的也必须是一个接口. 

　　注意invocationHandler的invoke()方法执行是在我们调用接口的方法的时候执行的.对于上面的代码就simplepost.getResponse()执行的时候.
