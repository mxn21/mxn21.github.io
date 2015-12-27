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
     
android:angle：（Integer） 渐变的角度，线性渐变时才有效，必须是45的倍数，0表示从左到右，90表示从下到上

android:centerX：（Float）渐变中心的相对X坐标，放射渐变时才有效，在0.0到1.0之间，默认为0.5，表示在正中间

android:centerY：（Float）渐变中心的相对X坐标，放射渐变时才有效，在0.0到1.0之间，默认为0.5，表示在正中间

android:centerColor ： （Color）中间点的色值

android:endColor ： （Color）结束的色值

android:startColor：（Color）开始的色值。

android:gradientRadius：（Float）渐变的半径，只有在android:type="radial"的时候有效。

android:type ：有三种类型
              Keyword.
              The type of gradient pattern to apply. Valid values are:
              
              "linear" A linear gradient. This is the default.  线性渐变，
              "radial" A radial gradient. The start color is the center color.  放射渐变，设置该项时，android:gradientRadius也必须设置
              "sweep" A sweeping line gradient.   扫描性渐变
 

android:useLevel
    Boolean.
    "true" if this is used as a LevelListDrawable.
    如果为true，将被当成LevelListDrawable使用。
    
    wwww
    
    389389
          





     
     
     
