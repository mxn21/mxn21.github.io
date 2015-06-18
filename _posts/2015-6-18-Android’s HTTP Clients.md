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


