---
layout: post
author: mxn
title: 为什么Application不能存储数据
category: 技术博文
tag: [gradle]
---

###不要在Application中存储数据

有一些数据可能会在app的很多地方使用到，也许是一些 session token，或者计算结果，等等。我们会偷懒不想把这些数据在activity之间不停地传递，或者保存在硬盘上。
所以有人为了让数据可以简单地跨越activity使用，就将数据存储在Application中，这个解决方案看上去简单，优雅，但是缺是完全错误的。
如果你认为你保存的数据会一直在Application中，那么程序会最终因为NullPointerException而崩溃。

###简单的测试

Application如下所示：
{% highlight java %}

class MyApplication extends Application {
    String name;
    String getName() {
        return name;
    }
    void setName(String name) {
        this.name = name;
    }
}
{% endhighlight %}

在第一个activity中，我们在Application中储存user的name属性。

{% highlight java %}
class WhatIsYourNameActivity extends Activity {
    void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.writing);
        MyApplication app = (MyApplication) getApplication();
        app.setName("Developer Phil");
        startActivity(new Intent(this, GreetLoudlyActivity.class));
    }
}
{% endhighlight %}

在第二个activity中，我们取出name

{% highlight java %}

class GreetLoudlyActivity extends Activity {
    TextView textview;
    void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.reading);
        textview = (TextView) findViewById(R.id.message);
    }
    void onResume() {
        super.onResume();
        MyApplication app = (MyApplication) getApplication();
        textview.setText("HELLO " + app.getName().toUpperCase());
    }
}

{% endhighlight %}

<!-- more -->

###剧本已经写好
1、用户打开了app。
2、在第一个activity中我们得到了name,然后储存在application中。
3、在第二个界面中我们从activity中取出name，然后展示在页面上。
4、用户按下了home键离开了app。
5、几个小时后，系统悄悄地杀死了app来释放内存。

到目前为止，没有出现什么问题，但是，接下来就到了程序崩溃的地方了。

6、用户重新打开了app。
7、系统重新创建了application实例，然后重新装载了第二个activity。
8、第二个页面去application中取name的值，但是现在是null,程序因为NullPointerException崩溃。

###程序崩溃的原因

在这个例子中，因为application重新实例化了一个新的，所以name的值是空，当我们调用String#toUpperCase() 的时候造成了NullPointerException。

那么核心问题就是：application不会永远在内存中，他会被杀死。
也就是说，如果想在后面的页面中使用前面的页面存储的数据，而且把数据存储在application中，那么你的程序是会可能崩溃的。

###解决方案

这里并没有一种神奇的方案，但是可以使用下面的其中一种方式。
1、把数据通过intent传递。
2、把数据保存在通过持久化保存，存储在文件或者数据库中等。
3、总是进行空指向检查，并且手动处理。

###如何模拟application被杀死










