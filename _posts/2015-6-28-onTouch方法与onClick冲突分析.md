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
