---
layout: post
author: mxn
titile: retrofit实现原理分析
category: 技术博文
tag: android
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
 
 使用了单例的PLATFORM，通过findPlatform()初始化实例， 由于retrofit支持不同的平台，Platform用于判断使用的是哪个平台，如果是Android平台就使用Platform.Android，retrofit的Android类继承了Platform, 根据android的特性对配置项做了处理.如果是Google AppEngine就使用Platform.AppEngine，否则使用Platform.Base，这些都是Platform的子类，其中AppEngine又是Base的子类。
Platform是一个抽象类，定义了以下几个抽象方法，这几个方法的作用就是返回一些RestAdapter中需要要用到成员的默认实现
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

所以上面的代码先对传入的Class 进行校验：Utils.validateServiceClass(service)，必须是接口，并且不能是一个没有自己函数并继承自其他父接口的空接口。
校验完成后返回一个动态代理类，我们想要知道retrofit使用用户自定义的接口干了什么事，就需要查看new RestHandler(getMethodInfoCache(service)) 的具体实现。

RestHandler 继承自 InvocationHandler 实现它的 invoke 函数，当代理类的接口函数被调用时，会先调用代理类的invoke 函数，然后在invoke 函数里通过反射调用用户指定的接口函数。

查看invoke 函数的具体实现之间，我们先分析分析getMethodInfoCache(service)函数。

下面是getMethodInfoCache函数的实现部分

{% highlight java %}
Map<Method, RestMethodInfo> getMethodInfoCache(Class<?> service) {
    synchronized (serviceMethodInfoCache) {
      Map<Method, RestMethodInfo> methodInfoCache = serviceMethodInfoCache.get(service);
      if (methodInfoCache == null) {
        methodInfoCache = new LinkedHashMap<Method, RestMethodInfo>();
        serviceMethodInfoCache.put(service, methodInfoCache);
      }
      return methodInfoCache;
    }
  }
 {% endhighlight  %}

该函数使用了同步代码块 synchronized 以serviceMethodInfoCache为对象锁，防止其他线程在该函数执行的时候入侵，之后在serviceMethodInfoCache查询service是否在其中存在，如果不存在，就新new 一个LinkedHashMap<Method, RestMethodInfo>加入其中，RestMethodInfo是一个final 类型的class，下面是它的申明
 
 {% highlight java %}
 /** Request metadata about a service interface declaration. */
final class RestMethodInfo {
final Method method;
  boolean loaded = false;   //方法是否已经load过(解析过)
  // Method-level details
  final ResponseType responseType;
  final boolean isSynchronous;      //方法是同步还是异步.
  final boolean isObservable;
  Type responseObjectType;
  RequestType requestType = RequestType.SIMPLE;
  String requestMethod;
  boolean requestHasBody;
  String requestUrl;           
  Set<String> requestUrlParamNames;
  String requestQuery;
  List<retrofit.client.Header> headers;
  String contentTypeHeader;
  boolean isStreaming;
  private enum ResponseType {  //方法的返回值是什么类型
    VOID,                      //void代表没有返回值,-->异步的方式
    OBSERVABLE,               //rxjava
    OBJECT                    //方法的返回值是对象--->同步的方式
  }
  RestMethodInfo(Method method) {
    this.method = method;
    responseType = parseResponseType();
    isSynchronous = (responseType == ResponseType.OBJECT);
    isObservable = (responseType ==           ResponseType.OBSERVABLE);
  }
   private ResponseType parseResponseType() {
    // Synchronous methods have a non-void return type.       //同步的方法有一个非void的返回值
    // Observable methods have a return type of Observable.   //Observable的方法 返回值类型应该是Observable
    Type returnType = method.getGenericReturnType();
    // Asynchronous methods should have a Callback type as the last argument. //异步的方法最后一个方法的参数类型应该是Callback.
    Type lastArgType = nul
    Class<?> lastArgClass = null;
    Type[] parameterTypes = method.getGenericParameterTypes();
    if (parameterTypes.length > 0) {
      Type typeToCheck = parameterTypes[parameterTypes.length - 1];
      lastArgType = typeToCheck;
      if (typeToCheck instanceof ParameterizedType) {
        typeToCheck = ((ParameterizedType) typeToCheck).getRawType();
      }
      if (typeToCheck instanceof Class) {
        lastArgClass = (Class<?>) typeToCheck;
      }
    }
    boolean hasReturnType = returnType != void.class;
    boolean hasCallback = lastArgClass != null && Callback.class.isAssignableFrom(lastArgClass); //如果有CallBack, CallBack只能是最后一个参数.
    // Check for invalid configurations.
    if (hasReturnType && hasCallback) {                         //返回值是非void类型和方法有CallBack参数有且只能有一个满足.
      throw methodError("Must have return type or Callback as last argument, not both.");
    }
    if (!hasReturnType && !hasCallback) {
      throw methodError("Must have either a return type or Callback as last argument.");
    }
    if (hasReturnType) {
      if (Platform.HAS_RX_JAVA) {
        Class rawReturnType = Types.getRawType(returnType);
        if (RxSupport.isObservable(rawReturnType)) {
          returnType = RxSupport.getObservableType(returnType, rawReturnType);
          responseObjectType = getParameterUpperBound((ParameterizedType) returnType);
          return ResponseType.OBSERVABLE;
        }
      }
      responseObjectType = returnType;
      return ResponseType.OBJECT;
    }
    lastArgType = Types.getSupertype(lastArgType, Types.getRawType(lastArgType), Callback.class);
    if (lastArgType instanceof ParameterizedType) {
      responseObjectType = getParameterUpperBound((ParameterizedType) lastArgType);
      return ResponseType.VOID;
    }
    throw methodError("Last parameter must be of type Callback<X> or Callback<? super X>.");
  }
private void parsePath(String path) {          //解析路径 如GET(string),注意这里会对string进行判断,必须以"/"开头. 所以不能是空字符串,如""
    ...........
}
  List<retrofit.client.Header> parseHeaders(String[] headers) {  //解析Headers注解.
　　.....
}
  private void parseParameters() {   //解析方法参数,如方法参数的@Header, @Query等.
　　.......
}
  private void parseMethodAnnotations() { //解析http方式.如GET(),POST()
　　.......
 }
}
{% endhighlight  %}

从RestMethodInfo类的注释来看，它和用户自定义的接口有密切的关系，也就是说，用户接口中定义的函数，由该类来解析说明和发出请求。

之后就将methodDetailsCache（LinkedHashMap<Method, RestMethodInfo>类型）传入到RestHandler类的构造方法当中，为其的同名属性完成赋值的同时返回该类的实例。接下来我们就开始分析RestHandler中的invoke 函数，这可以说是retrofit的精髓所在。

   {% highlight java %}
   private class RestHandler implements InvocationHandler {
   private final Map<Method, RestMethodInfo> methodDetailsCache;
   RestHandler(Map<Method, RestMethodInfo> methodDetailsCache) {  //一般的动态代理.InvocationHandler构造函数一般要传入委托类.
     this.methodDetailsCache = methodDetailsCache;
   }
   @SuppressWarnings("unchecked") //
   @Override public Object invoke(Object proxy, Method method, final Object[] args)  //动态代理的方法实现.调用委托类的方法,最终会调用invoke方法.
       throws Throwable {
     // If the method is a method from Object then defer to normal invocation.
     if (method.getDeclaringClass() == Object.class) {  //因为接口默认也继承Object.所以接口也有Object中的方法的. 如继承自Object如equals()方法
       return method.invoke(this, args);  //这里需要注意下,这里调用的是invocationHandler(H)的方法.一般的动态代理这里应该是调用委托类(A)的方法.
     }
     // Load or create the details cache for the current method.
     final RestMethodInfo methodInfo = getMethodInfo(methodDetailsCache, method);  //把method和RestMethodInfo缓存起来.防止重复解析method.(把方法的注解,返回值,同步异步等信息都解析出来储存在RestMethodInfo中.)
     if (methodInfo.isSynchronous) {  //同步的方式.
       try {
         return invokeRequest(requestInterceptor, methodInfo, args);  //核心方法(网络访问).   //可以看到retrofit的synchronous类型方法并不是子线程执行的. 所以在Android平台使用同步方式的retrofit的话要在子线程中.
       } catch (RetrofitError error) {
         Throwable newError = errorHandler.handleError(error);
         if (newError == null) {
           throw new IllegalStateException("Error handler returned null for wrapped exception.",  //自定义的ErrorHandler的hanleError()方法不能return null.
               error);
         }
         throw newError;  //注意,同步的方式,当遇到错误的时候,会抛出一个RuntimeException.在Android下尽量不要使用同步的方式(因为RuntimeException是不提示用户主动捕捉的).
       }
     }
     if (httpExecutor == null || callbackExecutor == null) {
       throw new IllegalStateException("Asynchronous invocation requires calling setExecutors.");
     }
     if (methodInfo.isObservable) {   //rx的方式
       if (rxSupport == null) {
         if (Platform.HAS_RX_JAVA) {
           rxSupport = new RxSupport(httpExecutor, errorHandler, requestInterceptor);
         } else {
           throw new IllegalStateException("Observable method found but no RxJava on classpath.");
         }
       }
       return rxSupport.createRequestObservable(new RxSupport.Invoker() {
         @Override public ResponseWrapper invoke(RequestInterceptor requestInterceptor) {
           return (ResponseWrapper) invokeRequest(requestInterceptor, methodInfo, args);
         }
       });
     }
     //同步, Observable方式都不是的话,肯定是异步的方式了.
     // Apply the interceptor synchronously, recording the interception so we can replay it later.
     // This way we still defer argument serialization to the background thread.
     final RequestInterceptorTape interceptorTape = new RequestInterceptorTape();
     requestInterceptor.intercept(interceptorTape);
     Callback<?> callback = (Callback<?>) args[args.length - 1];
     httpExecutor.execute(new CallbackRunnable(callback, callbackExecutor, errorHandler) {  //可以看到在Android上异步方式是通过HttpExecutor执行的.是默认子线程执行的. 所以异步方式的retrofit不需要在子线程中执行.
       @Override public ResponseWrapper obtainResponse() {
         return (ResponseWrapper) invokeRequest(interceptorTape, methodInfo, args);
       }
     });
     return null; // Asynchronous methods should have return type of void.
   }
   /**
    * Execute an HTTP request.
    *
    * @return HTTP response object of specified {@code type} or {@code null}.
    * @throws RetrofitError if any error occurs during the HTTP request.
    */
   private Object invokeRequest(RequestInterceptor requestInterceptor, RestMethodInfo methodInfo,
       Object[] args) {
     String url = null;
     try {
       methodInfo.init(); // Ensure all relevant method information has been loaded.  --->保证method的参数,返回值,注解都解析完成. 可以得到网络访问的url,header,query,访问方式等信息.
       String serverUrl = server.getUrl();
       RequestBuilder requestBuilder = new RequestBuilder(serverUrl, methodInfo, converter);
       requestBuilder.setArguments(args);
       requestInterceptor.intercept(requestBuilder);  //在网络访问之前,执行拦截器的方法. 保证拦截器是最后起作用的.
       Request request = requestBuilder.build();
       url = request.getUrl();
       if (!methodInfo.isSynchronous) {
         // If we are executing asynchronously then update the current thread with a useful name.
         int substrEnd = url.indexOf("?", serverUrl.length());
         if (substrEnd == -1) {
           substrEnd = url.length();
         }
         Thread.currentThread().setName(THREAD_PREFIX
             + url.substring(serverUrl.length(), substrEnd));
       }
       if (logLevel.log()) {
         // Log the request data.
         request = logAndReplaceRequest("HTTP", request, args);
       }
       Object profilerObject = null;
       if (profiler != null) {
         profilerObject = profiler.beforeCall();
       }
       long start = System.nanoTime();
       Response response = clientProvider.get().execute(request);  //调用okHttp或者其他Client进行网络访问.并把返回的数据封装进retrofit的Response中.
       long elapsedTime = TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - start);
       int statusCode = response.getStatus();                         //返回码
       if (profiler != null) {
         RequestInformation requestInfo = getRequestInfo(serverUrl, methodInfo, request);
         //noinspection unchecked
         profiler.afterCall(requestInfo, elapsedTime, statusCode, profilerObject);
       }
       if (logLevel.log()) {
         // Log the response data.
         response = logAndReplaceResponse(url, response, elapsedTime);
       }
       Type type = methodInfo.responseObjectType;
       if (statusCode >= 200 && statusCode < 300) { // 2XX == successful request   //返回码在200到300之间认为是一次成功的网络访问.
         // Caller requested the raw Response object directly.
         if (type.equals(Response.class)) {                       //如果方法的返回值类型(封装在responseObjectType)是retrofit的Response的话. 那就不用converter转换.
           if (!methodInfo.isStreaming) {
             // Read the entire stream and replace with one backed by a byte[].
             response = Utils.readBodyToBytesIfNecessary(response);
           }
           if (methodInfo.isSynchronous) {  //同步方式
             return response;         
           }
           return new ResponseWrapper(response, response); //异步的时候返回的是这个.
         }
         TypedInput body = response.getBody();
         if (body == null) {
           if (methodInfo.isSynchronous) {                   //① 结合下面的② ,返回值类型不同是因为invokeRequest()分别由2个不同的方法调用.  ①是由 invoke()调用的.
             return null;
           }
           return new ResponseWrapper(response, null);       //② 的情况是异步的. 是由callBack模式中的obtainResponse()中调用的.  下面和上面的返回值类型不同的情况皆是如此
         }
         ExceptionCatchingTypedInput wrapped = new ExceptionCatchingTypedInput(body);
         try {
           Object convert = converter.fromBody(wrapped, type); //方法的返回值是非Response. 那么由Converter把Response转换成对应的类型.
           logResponseBody(body, convert);
           if (methodInfo.isSynchronous) {
             return convert;
           }
           return new ResponseWrapper(response, convert);
         } catch (ConversionException e) {
           // If the underlying input stream threw an exception, propagate that rather than
           // indicating that it was a conversion exception.
           if (wrapped.threwException()) {
             throw wrapped.getThrownException();
           }
           // The response body was partially read by the converter. Replace it with null.
           response = Utils.replaceResponseBody(response, null);
           throw RetrofitError.conversionError(url, response, converter, type, e);
         }
       }
       response = Utils.readBodyToBytesIfNecessary(response);
       throw RetrofitError.httpError(url, response, converter, type);
     } catch (RetrofitError e) {
       throw e; // Pass through our own errors.
     } catch (IOException e) {
       if (logLevel.log()) {
         logException(e, url);
       }
       throw RetrofitError.networkError(url, e);
     } catch (Throwable t) {
       if (logLevel.log()) {
         logException(t, url);
       }
       throw RetrofitError.unexpectedError(url, t);
     } finally {
       if (!methodInfo.isSynchronous) {
         Thread.currentThread().setName(IDLE_THREAD_NAME);
       }
     }
   }
 }
 {% endhighlight  %}
 
 下面来分析每一步的调用流程。
（一）判断call的method是不是属于当前代理Object申明的，如果是就返回方法调用后的结果而不继续向下执行。

（二）调用getMethodInfo函数
传入参数为调用RestHandler构造方法时传入的methodDetailsCache（LinkedHashMap<Method, RestMethodInfo>类型）和具体调用的method(Method类型)，返回值是一个RestMethodInfo的实例。getMethodInfo函数主要作用是将用户调用的method解析为一个RestMethodInfo实例，然后再将method和该RestMethodInfo实例以key-value键值对的形式绑定，存放在methodDetailsCache中，然后再将该RestMethodInfo实例返回。（ps: RestMethodInfo中的对Mehod接口解析是很重要的，但是这里作者就一笔带过了，读者自己看看吧，就是JAVA反射那套东西，还是很简单的）

（三）调用并返回invokeRequest函数
这里拿到了将method解析后的methodInfo（RestMethodInfo类型），判断该调用函数用户是想让它同步执行还是异步执行（利用RestMethodInfo中的isSynchronous属性判断），同步执行则进入到if判断，调用并返回invokeRequest函数.

第一个参数RequestInterceptor接口是一个请求拦截器，能在请求执行之前向请求添加一些额外的数据，在最开始分析RestAdapter 创建的时候分析到ensureSaneDefaults 函数，如果没有在创建RestAdapter的时候传入自己实现的RequestInterceptor，那么Retrofit会给一个默认的RequestInterceptor空实现，指向RequestInterceptor接口类中的NONE属性。第二个参数是刚才Method解析后的RestMethodInfo，第三个参数是用户调用函数时传入的参数数组。

methodInfo.init() 
这个函数调用之后会将当前用户调用的方法进行进一步的解析，比如对@POST @GET @HEAD...etc 这些annotation的解析，并且每个实例只解析一次。
RequestBuilder requestBuilder = new RequestBuilder(serverUrl, methodInfo, converter);
从字面字面意思看，这个类和请求有关。确实，这个类里包含了用户请求网络所包含的各种参数，比如Host地址，Http协议头...etc 反正就是组成一个http请求所必要的东西。
Request request = requestBuilder.build();
由requestBuilder调用build函数产生一个Request类，该类是一个常量实体类，包含了最少的请求Http时所需要的信息。之后是请求信息的日志打印，在接下来关键的地方来了。
   {% highlight java %}
   long start = System.nanoTime();
   Response response = clientProvider.get().execute(request);
    long elapsedTime = TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - start);

 {% endhighlight  %}
 
 start 表示请求开始的时间，elapsedTime 表示该次请求花费的时间，而中间那句则是进行真正的网络请求，clientProvider 表示你在前面创建RestAdapter 时调调用ensureSaneDefaults函数所初始化的网络请求客户端（可能是OkClient, AndroidApacheClient,UrlFetchClient或者UrlConnectionClient四个的其中之一）。执行完成后返回一个response 这是一个Response常量实体类，和前面提到的Request常量实体类对应（一个表示请求实体，一个表示相应实体），里面存放着该次请求返回的Http协议头，状态码，body...etc .
 
 在请求之前还有一个profiler
   {% highlight java %}
 Object profilerObject = null;
        if (profiler != null) {
          profilerObject = profiler.beforeCall();
        }
  {% endhighlight  %}
  这里用到了AOP（面向切面）设计模式，如果用户设置了profiler的话，在请求的前后分别会调用profiler的beforeCall和afterCall函数，以通知用户请求的完成情况（在afterCall函数中传递了响应结果）。
之后的代码就是对response 的一个处理了，如果响应结果不在200到300之间，Retrofit会抛出一个自定义的异常进行进一步的处理，如果在200到300之间则会对response的请求数据做进一步的处理 .比如调用new ResponseWrapper(response, convert);利用converter（Converter接口类型，但实际上是GsonConverter，对Gson的一个封装） 将数据解析为用户想要返回的那个类型。

如果判断用户不是希望同步执行，那么invokeRequest函数的执行将会在之后执行。首先，会判断你是否使用了RxJava。如果使用了RxJava ，则组建一个RxSupport（对RxJava 中类的简单封装），使用它的createRequestObservable函数对invokeRequest函数进行异步调用并返回一个ResponseWrapper。否则使用httpExecutor（前面初始化的时候被设置为CachedThreadPool（缓存线程池））对invokeRequest函数进行异步调用并返回一个ResponseWrapper。

  最后看看CallbackRunnable的实现
    {% highlight java %}
    abstract class CallbackRunnable<T> implements Runnable {
  private final Callback<T> callback;
  private final Executor callbackExecutor;
  private final ErrorHandler errorHandler;
  CallbackRunnable(Callback<T> callback, Executor callbackExecutor, ErrorHandler errorHandler) {
    this.callback = callback;
    this.callbackExecutor = callbackExecutor;
    this.errorHandler = errorHandler;
  }
  @SuppressWarnings("unchecked")
  @Override public final void run() {
    try {
      final ResponseWrapper wrapper = obtainResponse();
      callbackExecutor.execute(new Runnable() {
        @Override public void run() {
          callback.success((T) wrapper.responseBody, wrapper.response);
        }
      });
    } catch (RetrofitError e) {
      Throwable cause = errorHandler.handleError(e);
      final RetrofitError handled = cause == e ? e : unexpectedError(e.getUrl(), cause);
      callbackExecutor.execute(new Runnable() {
        @Override public void run() {
          callback.failure(handled);
        }
      });
    }
  }
  public abstract ResponseWrapper obtainResponse();
} 
   {% endhighlight  %}
   
   就是一个普通的Runnable，在run方法中首先执行obtailResponse，从名字可以看到是执行请求返回Response，这个从前面可以看到执行了invokeRequest，和同步调用中一样执行请求。
紧接着就提交了一个Runnable至callbackExecutor，在看Platform时看到了callbackExecotor是通过Platform.get().defaultCallbackExecutor()返回的，Android中是向主线程的一个Handler发消息

值得注意的事，对于同步调用，如果遇到错误是直接抛异常，而对于异步调用，是调用Callback.failure()

### retrofit的大体流程

　用户自定义配置项的设置(如client,converter,拦截器等)--->解析接口的方法(如果曾经解析过就从缓存中获取),确定http访问的url,header,method等,确定是异步还是同步的方式------>使用具体的Client进行网络访问,并将数据封装到Response---->执行Converter的逻辑(有可能不用执行),把Response数据转换为一个具体对象.--->根据同步或者异步的方式,执行方法或者callBack的逻辑.
　
### retrofit框架的需要注意的几个小点.

1.为什么同步方式不像正常的方式一样要求用户try_catch来提醒用户捕捉异常?

　　通过上面的逻辑可以看到,真正进行网络访问,converter转换的逻辑都在invokeHandler.invoke()方法执行的时候执行. 而这个方法的调用是在 用户自定义接口调用接口方法的时候执行的.(不明白的可以看下动态代理的原理).而用户自定义的接口方法是没有抛出异常的.在java中,如果父类方法没有抛出异常,子类方法也不能显示的抛出异常.(子类方法只能抛出父类方法抛出异常或其子类).所以Retrofit就不能抛出各种异常(如IO异常). 并且要抓住异常后转换为RuntimeException抛出.(动态代理生成的接口的实现类其实内部也采用了同样的方法.)

　　异常抓住后不能直接内部处理,应该提醒用户代码执行的时候出了问题,所以必须抓住异常后再次抛出.而对于CallBack的方式,因为有failure()方法提示用户代码逻辑出了问题,所以就不用re-throw异常了.

　　2.关于 InvocationHandler的invoke()方法, 这个方法有个返回值. 那这个返回值返回的是什么呢?

　　首先明确Method.invoke(Object receiver,Object.. args)是和 receiver.method(args)等价的,2个方法的返回值是一样的.

public Object invoke(Object receiver, Object... args)  这里的Object是方法执行的的返回值.
---> Returns the result of dynamically invoking this method. Equivalent to {@code receiver.methodName(arg1, arg2, ... , argN)}.
　　动态代理生成了接口A的代理类B,B的同名方法内部其实调用的是invocationHandler的 invoke()方法.返回的也是invoke方法的返回值. 所以invoke返回的类型就应该和接口方法的返回值类型一样.

