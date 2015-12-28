---
layout: post
author: mxn
titile: GradientDrawable
category: 技术博文
tag: android
---

### GradientDrawable

用GradientDrawable实现渐变可以通过xml或者代码实现，xml实现需要在drawable下建立xml文件，在<shape>标签下建立<gradient>标签。

例如gradlient_background.xml文件如下：

      {% highlight xml  %}
      
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape android:shape="rectangle">
            <gradient android:startColor="#aa000000"
                      android:endColor="@android:color/transparent"
                      android:angle="90"
                />
        </shape>
    </item>
</selector>

     {% endhighlight %}
     
设置方法如下：

      {% highlight xml  %}
      
<RelativeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    >

    <!--<LinearLayout-->
        <!--android:layout_width="match_parent"-->
        <!--android:layout_height="200dp"-->
        <!--android:background="@drawable/gradlient_background"-->
        <!--/>-->

    <LinearLayout
        android:id="@+id/layout_bottom"
        android:layout_width="match_parent"
        android:layout_height="300dp"
        android:layout_alignParentBottom="true"
        android:background="@drawable/gradlient_background"
        android:orientation="horizontal"
        />

</RelativeLayout>  
        
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
          





     
     
     
