---
layout: post
author: mxn
titile: 深入了解android view绘制机制
category: 技术博文
tag: android
---

这篇文章讲解View的工作原理以及自定义View的方法

#### LayoutInflater

先来看一下LayoutInflater的基本用法吧，它的用法非常简单，首先需要获取到LayoutInflater的实例，有两种方法可以获取到，第一种写法如下：

    {% highlight java  %}
    LayoutInflater layoutInflater = LayoutInflater.from(context);
    {% endhighlight %}

当然，还有另外一种写法也可以完成同样的效果：

    {% highlight java  %}
    LayoutInflater layoutInflater = (LayoutInflater) context
        .getSystemService(Context.LAYOUT_INFLATER_SERVICE);

        {% endhighlight %}

其实第一种就是第二种的简单写法，只是Android给我们做了一下封装而已。得到了LayoutInflater的实例之后就可以调用它的inflate()方法来加载布局了，如下所示：
layoutInflater.inflate(resourceId, root);

inflate()方法一般接收两个参数，第一个参数就是要加载的布局id，第二个参数是指给该布局的外部再嵌套一层父布局，如果不需要就直接传null。这样就成功成功创建了一个布局的实例，之后再将它添加到指定的位置就可以显示出来了。


