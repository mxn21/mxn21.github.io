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
