---
layout: post
author: mxn
titile: PathMeasure使用方法
category: 技术博文
tag: android
---

沿预定路径动画图像（无论是直线或曲线）是做动画时一种常见的情况。传统方法是寻找path的函数，带入x的值来计算y的值，
然后使用坐标(x,y)来完成动画。我们可以在一个更简单的方法，就说通过使用path和pathmeasure。

假设，我想在一条直线上移动图像，从（0,0）到（100，100）。

下面是一个代码段，告诉你如何做：
声明我们将使用的变量：

    {% highlight java %}

Path path;
PathMeasure measure;
float[] pos, tan;
float speed, distance;

    {% endhighlight %}

初始化path对象和其他变量，我们将在后面使用。如果使用SurfaceView，你可以在你的surfaceCreated()回调中执行这些。

<!-- more -->

    {% highlight java %}

// Init the Path.
 path=new Path();

// Set the starting position of the path to (0,0).
 path.moveTo(0,0);

// Add a line to the Path, starting from (0,0), ending at (100, 100).
 path.lineTo(100,100);

// Create a PathMeasure object, passing in the Path object
 // we created and a boolean that specifies if the Path should
 // be forced to a closed path.
 measure = new PathMeasure(path, false);

// Here, we're dividing the whole length of the path by 30.
 speed = measure.getLength() / 30;


pos=new float[2];
 tan=new float[2];

    {% endhighlight %}









PathMeasure主要是配合Path，来计算Path里面点的坐标的，或者是给一个范围，来截取Path其中的一部分的。






