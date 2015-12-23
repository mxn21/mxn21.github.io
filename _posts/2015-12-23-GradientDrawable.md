---
layout: post
author: mxn
titile: GradientDrawable
category: 技术博文
tag: android
---

<shape>标签

      {% highlight java  %}
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

      {% highlight java  %}
<TextView  
        android:id="@+id/textView"  
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
        android:background="@drawable/gradient_box"  
        android:text="测试" />  
        
        {% endhighlight %}