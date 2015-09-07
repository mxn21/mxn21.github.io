---
layout: post
author: mxn
titile: Closeable和Flushable接口
category: 技术博文
tag: android
---

Closeable和Flushable接口在JDK 1.5 version中被定义，在java.io 包中。

### Closeable接口

Closeable接口包含唯一一个抽象方法close()，当close()被调用时，stream对象持有的资源被释放，也可以在程序的其他地方调用
以避免内存泄露。很多的stream classes继承了Closeable接口并且覆写了close()。任何继承这个接口的类都可以调用close()来
关闭流的操作。当然如果父类继承了这个接口，子类同样可以调用close()。例如InputStream继承了这个接口，那么他的子类FileInputStream
也可以使用close()。由于InputStream, OutputStream, Reader,Writer都继承了这个接口，他们所有的子类都可以调用close()。


### Flushable接口

Flushable接口也只包含一个方法flush()。很多输出流继承了这个接口，并且覆写了flush()。当这个方法被调用时，缓存中的数据被写入到文件中。


下面是一下继承了Closeable接口或Flushable接口的类的例子：

    {% highlight java  %}

    public abstract class InputStream extends Object implements Closeable
    public abstract class OutputStream extends Object implements Closeable, Flushable
    public abstract class Reader extends Object implements Readable, Closeable
    public abstract class Writer extends Object implements Appendable, Closeable, Flushable
    public class PrintStream extends FilterOutputStream implements Appendable, Closeable

     {% endhighlight %}

close()和flush()方法都抛出了异常exception IOException，在使用时必须要处理异常。


### Appendable

在说一下Appendable接口，继承这个接口的所有已知实现类：BufferedWriter, CharArrayWriter, CharBuffer, FileWriter,
FilterWriter, LogStream, OutputStreamWriter, PipedWriter, PrintStream, PrintWriter, StringBuffer,
StringBuilder, StringWriter, Writer。

Appendable接口的实现类的对象能够被添加 char 序列和值。如果某个类的实例打算接收取自 java.util.Formatter 的格式化输出，那么该类必须实现 Appendable 接口。
要添加的字符应该是有效的 Unicode 字符。Appendable 对于多线程访问而言是安全的。线程安全由扩展和实现此接口的类负责。

Appendable有三个方法：

    {% highlight java  %}

    //向此 Appendable 添加指定字符。
    Appendable append(char c) throws IOException;
    //向此 Appendable 添加指定的字符序列。
    Appendable append(CharSequence csq) throws IOException;
    //向此 Appendable 添加指定字符序列的子序列。
    Appendable append(CharSequence csq, int start, int end) throws IOException;

     {% endhighlight %}

