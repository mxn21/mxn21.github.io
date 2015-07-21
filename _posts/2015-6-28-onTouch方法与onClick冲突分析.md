---
layout: post
author: mxn
title: onTouch方法与onClick冲突分析
category: 技术博文
tag: [android]
---

下面分析控件本身的onTouch和onClick方法的执行冲突

首先自定义一个MyLayout继承LinearLayout，重写dispatchTouchEvent，onInterceptTouchEvent，onTouchEvent方法，代码如下:

    {% highlight java  %}
    public class MyLayout extends LinearLayout {

        @Override
        public boolean dispatchTouchEvent(MotionEvent ev) {
            Log.e("Demo:","父View的dispatchTouchEvent方法执行了");
            return super.dispatchTouchEvent(ev);
         }

        @Override
        public boolean onInterceptTouchEvent(MotionEvent ev) {
            Log.e("Demo:","父View的onInterceptTouchEvent方法执行了");
            return super.onInterceptTouchEvent(ev);
        }

        @Override
        public boolean onTouchEvent(MotionEvent event) {
            Log.e("Demo:","父View的onTouchEvent方法执行了");
            return super.onTouchEvent(event);
        }

        public MyLayout(Context context) {
            super(context);
        }

        public MyLayout(Context context,AttributeSet attr) {
            super(context,attr);
        }
    }

    {% endhighlight %}

在这三个方法中打印一段信息，有助于查看这三个方法的执行流程

下面是自定义一个MyTextView继承TextView，重写dispatchTouchEvent，onTouchEvent方法(这里没有onInterceptTouchEvent方法，
原因很简单，就是TextView是一个View没有子View了，不需要拦截信息了)代码如下:

    {% highlight java  %}
    public class MyTextView extends TextView{

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
    Log.e("Demo:","子View的dispatchTouchEvent方法执行了");
    return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
    Log.e("Demo:","子View的onTouchEvent方法执行了");
    return super.onTouchEvent(event);
    }

    public MyTextView(Context context) {
    super(context);
    }

    public MyTextView(Context context, AttributeSet attrs) {
    super(context, attrs);
    }

    public MyTextView(Context context, AttributeSet attrs,int defStyle) {
    super(context, attrs, defStyle);
    }
    }
        {% endhighlight %}

在方法中输出log信息，便于查看方法的执行流程

<!-- more -->

定义的一个布局文件:

    {% highlight xml  %}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">
        <com.bbdtek.demo.MyLayout
            android:id="@+id/layout"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            android:orientation="vertical">
            <com.bbdtek.demo.MyButton
                android:id="@+id/btn"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="点我"/>
            <com.bbdtek.demo.MyTextView
                android:id="@+id/txt1"
                android:layout_width="fill_parent"
                android:layout_height="wrap_content"
                android:text="hello world"
                android:textSize="25dp"/>
        </com.bbdtek.demo.MyLayout>
</LinearLayout>
            {% endhighlight %}

MyLayout中包含一个MyTextView控件:

下面来看一下测试代码:

    {% highlight java  %}
    public class AndroidDemoActivity extends Activity{

        public void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.main);
           MyTextView txt1 = (MyTextView)findViewById(R.id.txt1);
           //MyLayout layout = (MyLayout)findViewById(R.id.layout);
           txt1.setOnClickListener(new OnClickListener(){
    @Override
    public void onClick(View v) {
    Log.e("Demo:","点击了txt1");
    }});
           //给MyTextView定义一个onTouchListener监听器
           txt1.setOnTouchListener(new OnTouchListener(){
        @Override
        public boolean onTouch(View v, MotionEvent event) {
        if(event.getAction() == MotionEvent.ACTION_DOWN){
        Log.e("Demo:","txt1按下");
        }else if(event.getAction() == MotionEvent.ACTION_UP){
        Log.e("Demo:","txt1弹起");
        }
        return false;
        }});
           //给MyLayout定义一个onTouchListener监听器
           /*layout.setOnClickListener(new OnClickListener(){
    @Override
    public void onClick(View v) {
    Log.e("Demo:","点击了父layout");
    }});*/

        }
    }
     {% endhighlight %}

