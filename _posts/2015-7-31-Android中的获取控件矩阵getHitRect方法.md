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

