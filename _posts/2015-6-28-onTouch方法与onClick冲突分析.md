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

看一下运行结果:

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img30.png)

可以看到先执行的是MyTextView的dispatchTouchEvent方法(这个方法是每次触发onTouch方法都会执行的)
，然后是执行了onTouch方法中的ACTION_DOWN中的代码，然后执行了MyTextView中的onTouchEvent方法，
当用户弹起手指的时候又一次执行了这样的一个过程，最后就是执行了onClick方法，在这里就来看一下onTouchEvent中的源代码:

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img31.png)

因为TextView继承View，可以查看View中的onTouchEvent方法，在这个方法中又调用了ViewGroup中的onTouchEvent方法，
下面在来看一下ViewGroup中的onTouchEvent方法:

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img32.png)


在ACTION_UP中的这段代码就是执行了onClick方法，具体可以看一下performClick方法解释:

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img33.png)

mOnClickListener就是OnClickListener监听器，执行了onClick方法，所以上面的onClick方法是在ACTION_UP之后执行了。
下面在来看一下这种情况，现在把onTouch方法的返回值改成true：看一下执行结果:

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img34.png)

可以看到首先还是执行了MyTextView的dispatchTouchEvent方法，然后执行了onTouch中的ACTION_DOWN代码，同样当用户的弹起手指的时候执行了同样的过程，现在的问题是这样的执行过程为什么和上面的不一样呢？
首先来看一下有哪些不一样的地方：
第一个不一样的地方就是”子View的onTouchEvent方法执行了“这句话没有执行，那就是MyTextView中的onTouchEvent方法没有执行了，来看一下源码:

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img35.png)

在View中的dispatchTouchEvent方法中可以看到，是先执行OnTouchListener监听器中的onTouch方法，
如果onTouchListener不为null,并且onTouch方法返回false的时候才执行onTouchEvent方法，
现在我们把onTouch方法的返回值变成了true，所以onTouchEvent方法就不执行了。这里一定要注意onTouchEvent方法和onTouch方法的区别。

第二个不一样的地方就是onClick方法没有执行了，因为onClick是在onTouchEvent执行后执行的，而 onTouchEvent都没有执行，所以onClick更不会执行。
所以如果onTouch方法返回true，说明View这次消费了这次事件，所以就不会再执行后续的onClick方法了。

所以事件传递流程是 onTouch --> onTouchEvent --> onClick。onTouch如果返回true,后面都不会执行。


这里再解释一下为什么ListView中的Item中如果有Button，CheckBox等这样的组件的话，ListView中的setOnItemClick方法就是失效了，
原因是Item没有获取焦点，焦点被Button等控件默认获取到了。ListView无法获得焦点，当然不能响应setOnItemClick。源码如下：





