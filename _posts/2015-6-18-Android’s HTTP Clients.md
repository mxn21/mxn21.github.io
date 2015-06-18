---
layout: post
author: mxn
title: Android’s HTTP Clients
category: 技术博文
tag: [android]
---

android包含了两种HTTP clients： HttpURLConnection 和 Apache HTTP Client。这两种都支持HTTPS,上传下载流, 可配置的链接时间, IPv6 和线程池。

### Apache HTTP Client

DefaultHttpClient 和它的同类AndroidHttpClient都是适合浏览网页的可扩展的HTTP clients.他们有庞大且灵活的APIs.使用很稳定，而去很少有bug.
但是他们庞大的api使我们很难在不破坏兼容性的情况下对他们升级。android的开发团队已经不在继续使用 Apache HTTP Client.
AndroidHttpClient默认不带gzip压缩。
和HttpURLConnection相比两种方式请求connection都是keep alive，但是默认User-Agent不同。
![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img4.jpg)

### HttpURLConnection
Java的HttpURLConnection，默认带gzip压缩
![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img3.jpg)
HttpURLConnection是一种很适合的轻量级 HTTP client，他的核心api可以让我们很简单的进行升级。
但是在android 2.2之前HttpURLConnection有一些失败的bug.特别是在可读输入流里调用 close()的时候，会影响链接池，所以只能禁用连接池。

{% highlight java %}
private void disableConnectionReuseIfNecessary() {
    // HTTP connection reuse which was buggy pre-froyo
    if (Integer.parseInt(Build.VERSION.SDK) < Build.VERSION_CODES.FROYO) {
        System.setProperty("http.keepAlive", "false");
    }
}
{% endhighlight  %}
在android 2.3 Gingerbread中增加了gzip支持 HttpURLConnection会在发出的request中自动增加这些header，并且处理对应的response:
Accept-Encoding: gzip

 <!-- more -->
 在android 4.0 Ice Cream Sandwich 中也进行了一些升级，HttpURLConnection可以链接同一个ip的多个站点（既SNI，Server Name Indication），同时支持压缩和session.如何链接失败，会自动重连。HttpsURLConnection可以在保存旧服务器的兼容心的情况下，更高效的链接新的服务器 。 

在android 4.0中增加了response缓存，当使用cache时，HTTP request和满足一下三种情况之一：
1、完全缓存responses到本地磁盘，当没有新的链接需要是，可以立即使用本地缓存。
2、有选择的缓存一部分内容，需要服务器做一些验证。客户端发送一个request，例如：“Give me /foo.png if it changed since yesterday”
服务器返回更新的内容，或者304 Not Modified status。如果没有更新，则没有内容被下载。
3、不使用缓存，responses会在以后需要时再缓存。
 
可以使用反射来缓存response到本地，下面的示例代码会在不影响之前版本的情况下打开缓存：
{% highlight java %}
private void enableHttpResponseCache() {
    try {
        long httpCacheSize = 10 * 1024 * 1024; // 10 MiB
        File httpCacheDir = new File(getCacheDir(), "http");
        Class.forName("android.net.http.HttpResponseCache")
            .getMethod("install", File.class, long.class)
            .invoke(null, httpCacheDir, httpCacheSize);
    } catch (Exception httpResponseCacheNotAvailable) {
    }
}
{% endhighlight  %}

同时还要在服务器HTTP responses中设置cache headers。

### 哪个是最好的client
在 Froyo(2.2) 之前，HttpURLConnection 有个重大 Bug，调用 close() 函数会影响连接池，导致连接复用失效，所以在 Froyo 之前使用 HttpURLConnection 需要关闭 keepAlive。
 
另外在 Gingerbread(2.3) HttpURLConnection 默认开启了 gzip 压缩，提高了 HTTPS 的性能，Ice Cream Sandwich(4.0) HttpURLConnection 支持了请求结果缓存。
再加上 HttpURLConnection 本身 API 相对简单，所以对 Android 来说，在 2.3 之后建议使用 HttpURLConnection，之前建议使用 AndroidHttpClient。
 
Retrofit及Volley框架默认在Android Gingerbread(API 9)及以上都是用HttpURLConnection，9以下用HttpClient。

### GZip压缩

一般对于API请求需带上GZip压缩，因为API返回数据大都是JSon串之类字符串，GZip压缩后内容大小大幅降低，下面是这两个网页GZip压缩前后对比，都是第一条表示GZip压缩后，第二条为压缩前

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img5.jpg)

