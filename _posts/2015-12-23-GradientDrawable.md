---
layout: post
author: mxn
titile: GradientDrawable
category: 技术博文
tag: android
---


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