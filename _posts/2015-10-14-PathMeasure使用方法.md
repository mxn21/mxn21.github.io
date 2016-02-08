---
layout: post
author: mxn
titile: PathMeasure使用方法
category: 技术博文
tag: android
---

沿预定路径动画图像（无论是直线或曲线）是做动画时一种常见的情况。传统方法是寻找path的函数，带入x的值来计算y的值，
然后使用坐标(x,y)来完成动画。我们可以在一个更简单的方法，就说通过使用path和pathmeasure。

重点看下pathmeasure常用方法：

* float getLength() 返回当前contour的长度，也就是这一个Path有多长.
* boolean getPosTan(float distance, float[] pos, float[] tan) 传入一个距离distance(0<=distance<=getLength())，
然后会计算当前距离的坐标点和切线，注意，pos会自动填充上坐标，这个方法很重要.
* boolean getSegment(float startD, float stopD, Path dst, boolean startWithMoveTo) 传入一个开始和结束距离，
然后会返回介于这之间的Path，在这里就是dst，他会被填充上内容，这个方法很重要.
* boolean nextContour() 移动到下一个笔画，如果你的Path是由多个笔画组成的话，那么就可以使用这个方法.
* void setPath(Path path, boolean forceClosed)这个方法也比较重要，用来设置新的Path对象的，算是对第一个构造函数的一个补充.

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

后面的图像移动的概念很简单：从（0,0）开始，在每一个更新周期，很小的一段时间内，我们找到沿path的坐标点。
现在在刷新画面时，我们做下面的事：

    {% highlight java %}
public void update()
 {
 while(distance < measure.getLength())
 {
 // getPosTan pins the distance along the Path and
 // computes the position and the tangent.
 measure.getPosTan(distance, pos, tan);
 pointArray[counter] = new FlaotPoint(pos[0],
                    pos[1]);
  counter++;
  distance += speed;   // Traversal
 }
 }
    {% endhighlight %}

现在来渲染图片：

    {% highlight java %}
public void render(Canvas canvas)
{
//Draw the bitmap on the canvas, passing in the
//Bitmap object, the x and y co-ordinate and a
// Paint object.
canvas.render(bmpImage, pos[0], pos[1], null);
}
    {% endhighlight %}

这里的speed直接决定了动画的速度。调整这个值，你可以到达你想要的速度。
就像一条线一样，你也可以使用曲线，贝赛尔曲线，弧线，和这些的组合等等。

下面再看一个具体的例子：

已知图中的中心圆点在屏幕上的坐标为(x, y)，分别求出点1、2、3、4的坐标值。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img115.png)

解决方法：
可以利用 android.graphics.Path和android.graphics.PathMeasure，
1.以圆点坐标（x，y）为中心画一个矩形RectF，
2.再通过Path类画一个90度（180—270）的内切圆弧路径，
3.然后将该路径平分成3段，
4.再利用PathMeasure分别测量出各个点的坐标值


先简单介绍一下Path类的一些方法的使用和说明

1.addArc(RectF oval, float startAngle, float sweepAngle) 画扇形（弧线）

第一个参数是一个矩形，第二个参数为0时的位置是矩形右边1/2高度的点，90为矩形底部1/2宽的位置，正数为顺时针旋转，负数是逆时针旋转。
第三个参数是图形绘制角度，下图第三个参数为180，如果是-180，那么图形倒过来。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img116.png)

2.addCircle(float x, float y, float radius, Path.Direction dir) 画圆

第一、二参数是圆心坐标，第三参数是半径，第四参数是顺时针画还是逆时针画.

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img117.png)

3.addOval(RectF oval, Path.Direction dir) 画椭圆.

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img118.png)

4.addPath(Path src, float dx, float dy)

复制一份Path，包含被复制的src的一切，并向X与Y轴方向移动第二、三参数的距离。

5.addRect(RectF rect, Path.Direction dir)
画个矩形

6.addRect(float left, float top, float right, float bottom, Path.Direction dir)
画个矩形、四个参数对应与原点的相对距离的四个点。

7.addRoundRect(RectF rect, float rx, float ry, Path.Direction dir)

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img119.png)

画圆角矩形。第二、三个参数为0时就是个矩形，为360时，就是个椭圆。第二个参数指X轴方向的角度，决定了与参考矩形的横线交点位置，
0-360决定交点范围为 角点与线中点之间的某点。

8.arcTo(RectF oval, float startAngle, float sweepAngle)
等同于arcTo(RectF oval, float startAngle, float sweepAngle, boolean false)。
从之前的最后一点开始画线到画椭圆的开始点，接着画个椭圆。

9.arcTo(RectF oval, float startAngle, float sweepAngle, boolean forceMoveTo)
如果最后一个参数为true，那么等同于addArc(RectF oval, float startAngle, float sweepAngle)。

10.cubicTo(float x1, float y1, float x2, float y2, float x3, float y3)
画贝塞尔曲线。前四个参数是两个控制点，最后俩个参数是终止点。起始点通过moveTo(float x, float y)或者setLastPoint(float dx, float dy)方法设置。关于贝塞尔曲线，可以去网上找找资料。某人的博客，关于此曲线。

11.moveTo(float x, float y)
设置下一个图形的开始点。

12.setLastPoint(float dx, float dy)
设置图形的最后一个点位置。如果画的是个封闭图形，而这个点不在图形线上，那么这个点与最后一个图形连上线完成封闭。
如图，本来画了个圆角矩形，最后setLastPoint了一下。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img120.png)

13.close()
关闭当前图形，如果最后一点不是开始的那点，那么从最后一点画线到开始点。简而言之，画三角型只需要画俩条线，再调此方法能三角型就完成了。

下面实现前面的需求，求3个点的坐标代码如下：

    {% highlight java %}
private void calculateItemPositions() {
        // Create an arc that starts from startAngle and ends at endAngle
        // in an area that is as large as 4*radius^2
        //获取中心圆点的坐标值
        Point center = getActionViewCenter();
        //内切弧形路径
        //以圆点坐标（x，y）为中心画一个矩形RectF
        RectF area = new RectF(center.x - radius, center.y - radius, center.x + radius, center.y + radius);
        Path orbit = new Path();
        //通过Path类画一个90度（180—270）的内切圆弧路径
        orbit.addArc(area, startAngle, endAngle - startAngle);
        PathMeasure measure = new PathMeasure(orbit, false);
        // Prevent overlapping when it is a full circle
        //然后将该路径平分成3段，这里的size为4
        int divisor;
        if(Math.abs(endAngle - startAngle) >= 360 || subActionItems.size() <= 1) {
            divisor = subActionItems.size();
        }
        else {
            divisor = subActionItems.size() -1;
        }
        // Measure this path, in order to find points that have the same distance between each other
        for(int i=0; i<subActionItems.size(); i++) {
            float[] coords = new float[] {0f, 0f};
            //利用PathMeasure分别测量出各个点的坐标值coords
            measure.getPosTan((i) * measure.getLength() / divisor, coords, null);
            // get the x and y values of these points and set them to each of sub action items.
            subActionItems.get(i).x = (int) coords[0] - subActionItems.get(i).width / 2;
            subActionItems.get(i).y = (int) coords[1] - subActionItems.get(i).height / 2;
        }
    }
    {% endhighlight %}

