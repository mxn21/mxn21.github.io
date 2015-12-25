---
layout: post
author: mxn
titile: GradientDrawable
category: 技术博文
tag: android
---

<shape>标签

      {% highlight xml  %}
<?xml version="1.0" encoding="utf-8"?>  
<shape xmlns:android="http://schemas.android.com/apk/res/android"  
    android:shape="rectangle">  
    <gradient  
        android:startColor="#FFFF0000"  
        android:endColor="#80FF00FF"  
        android:angle="45"/>  
    <padding android:left="7dp"  
        android:top="7dp"  
        android:right="7dp"  
        android:bottom="7dp" />  
    <corners android:radius="8dp" />  
</shape>
     {% endhighlight %}
     
<!-- more -->

      {% highlight xml  %}
<TextView  
        android:id="@+id/textView"  
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
        android:background="@drawable/gradient_box"  
        android:text="测试" />  
        
        {% endhighlight %}
   
         {% highlight java  %}
TextView textView=(TextView)findViewById(R.id.textView);  
ShapeDrawable gradientDrawable=(ShapeDrawable)textView.getBackground();  
   
        {% endhighlight %}
        
不能将GradientDrawable转换为ShapeDrawable。
可见，使用<shape>标签定义的是GradientDrawable
。

设置背景色可以通过在res/drawable里定义一个xml,如下：

     {% highlight xml  %}
<?xml version="1.0" encoding="utf-8"?>  
<shape xmlns:android="http://schemas.android.com/apk/res/android">  
      <gradient
            android:angle="90"
            android:centerColor="#FF3f3f3f"
            android:centerY="0.5"
            android:endColor="#FF2c2c2c"
            android:startColor="#FF2c2c2c"
            android:type="linear"
            android:useLevel="false" /> 
</shape>  
     {% endhighlight %}
     
android:angle：（Integer） 0是从左到右渐变，90是从下到上渐变。要45的整数倍，说明是可以斜着渐变的。

android:centerX：（Float）X轴渐变中心点的位置（0 到1 的一个值）

android:centerY：（Float）Y轴渐变中心点的位置 （0 到1 的一个值）


     
     
     
