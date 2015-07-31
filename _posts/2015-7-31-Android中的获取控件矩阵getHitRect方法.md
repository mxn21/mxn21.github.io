---
layout: post
author: mxn
titile: Android中的获取控件矩阵getHitRect方法
category: 技术博文
tag: android
---

### getHitRect的作用和用法

public  void getHitRect(Rect outRect)这个方法用来找到控件占据的矩形区域的矩形坐标。
参数outRect表示控件占据的矩形区域。

测试代码如下，根据代码可以很直观的知道这个函数的用法：

  {% highlight java  %}
public class MainActivity extends Activity {
    private TextView textView = null;
    private Button button = null;

    Rect Trect = new Rect();
    Rect Brect = new Rect();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        textView = (TextView) findViewById(R.id.tv01);
        button = (Button) findViewById(R.id.btn);
        button.setOnClickListener(new OnClickListener() {
            public void onClick(View v) {
                textView.getHitRect(Trect);
                button.getHitRect(Brect);
                Log.e("TAG", "---------TextView_left-------" + String.valueOf(Trect.left));
                Log.e("TAG", "---------TextView_top-------" + String.valueOf(Trect.top));
                Log.e("TAG", "---------TextView_right-------" + String.valueOf(Trect.right));
                Log.e("TAG", "---------TextView_bottom-------" + String.valueOf(Trect.bottom));
                Log.e("TAG", "---------Button_left-------" + String.valueOf(Brect.left));
                Log.e("TAG", "---------Button_top-------" + String.valueOf(Brect.top));
                Log.e("TAG", "---------Button_right-------" + String.valueOf(Brect.right));
                Log.e("TAG", "---------Button_bottom-------" + String.valueOf(Brect.bottom));
            }
        });
    }
}
     {% endhighlight %}

<!-- more -->

查看log如下：

  {% highlight java  %}
06-17 18:07:55.534: E/hailong(5797): ---------TextView_left-------281
06-17 18:07:55.534: E/hailong(5797): ---------TextView_top-------475
06-17 18:07:55.534: E/hailong(5797): ---------TextView_right-------438
06-17 18:07:55.534: E/hailong(5797): ---------TextView_bottom-------513
06-17 18:07:55.534: E/hailong(5797): ---------Button_left-------281
06-17 18:07:55.534: E/hailong(5797): ---------Button_top-------513
06-17 18:07:55.534: E/hailong(5797): ---------Button_right-------439
06-17 18:07:55.534: E/hailong(5797): ---------Button_bottom-------609
     {% endhighlight %}

这样我们就可以知道控件的顶点坐标了。

可以用类似的方法判断是否点击到该控件上boolean isHit = Trect.contains((int)event.getX(), (int)event.getY());


注意：getHitRect作为获取控件所在的矩阵范围函数，我们平常调用时候如果是在控件的监听器里调用就没事，
但是如果主动的在onCreate 或者 onResume中，拿到的矩阵坐标全是0.因为getHitRect方法不能直接在onCreate中调用
，原因是该控件还未在这个界面框架中得以测量布局，不知道到底是多少，所以我们要寻找一个时机去做这件事，两个方法：

1.运用该控件执行post（就把下面的那个parent当成你要获取getHitRect的方法）

  {% highlight java  %}
final View parent = (View) delegate.getParent();
parent.post( new Runnable() {
    // Post in the parent's message queue to make sure the parent
    // lays out its children before we call getHitRect()
    public void run() {
        final Rect r = new Rect();
        delegate.getHitRect(r);
        r.top -= 4;
        r.bottom += 4;
        parent.setTouchDelegate( new TouchDelegate( r , delegate));
    }
});
     {% endhighlight %}

2.自定义该控件，覆写onDraw，调用getHitRect.


### 扩大view点击范围

Android4.0设计规定的有效可触摸的UI元素标准是48dp，转化为一个物理尺寸约为9毫米。7~10毫米，这是一个用户手指能准确并且舒适触摸的区域。

如下图所示，你的UI元素可能小于48dp，图标仅有32dp，按钮仅有40dp，但是他们的实际可操作焦点区域最好都应达到48dp的大小。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img53.jpg)


