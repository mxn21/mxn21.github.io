---
layout: post
author: mxn
titile: clipChildren属性分析
category: 技术博文
tag: android
---

clipChildren属性和clipToPadding属性类似，先比较一个两个属性。

1.clipChildren用来定义他的子控件是否要在他应有的边界内进行绘制。 默认情况下，clipChildren被设置为true。 也就是不允许进行扩展绘制。

2.clipToPadding用来定义ViewGroup是否允许在padding中绘制。默认情况下，cliptopadding被设置为ture， 也就是把padding中的值都进行裁切了。

这两个属性联合起来能干什么呢？用来做一些类似于点击放大等点击特效非常合适.不用去更改布局，只需加入这两个属相，并引入动画效果就完成了。

ClipChildren属性对需要做动画的View非常有用，特别是对ScaleAnimation。
需要注意的是ClipChildren ＝ false只是告诉他的子View可以超出子view本身的大小，并不是说他自己可以超出他的父View的大小，
所以如果你想允许某个View超过其父View的边界，你需要一直找到最顶层的父View并设置ClipChildren = false。

下面需要提一下的是一个关于ClipChildren的坑，在4.2的系统上(4.3未测试，4.4及5.0版本没有这个 BUG)，在启用了hardware accelerated的情况下，
ScaleAnimation会出现撕裂等不流畅的现象，处理方法如下：

    {% highlight java %}
if (Build.VERSION.SDK_INT < Build.VERSION_CODES.KITKAT) {
    parentView.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
 }
    {% endhighlight %}

这里的parentView一定要是你启用了ClipChildren = false的那个ViewGroup，否则没有效果，最佳实践是只在需要做ScaleAnimation的时候才关闭硬件加速，做完动画后应该重新开启。

<!-- more -->

看看最后的效果吧， 点击第三个机器人就会播放一个变大的效果（类似于心变大的效果）

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img114.png)

布局代码

主要看设的两个关键属性android:clipChildren和android:clipToPadding均为false。
这就让点击第三个小人时，可以跨边界进行绘制，并且允许其在padding区域内绘制。

    {% highlight xml %}

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context=".MainActivity"
    android:clipChildren="false"
    android:clipToPadding="false">

    <ImageView
        android:id="@+id/img1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:src="@drawable/ic_launcher" />

    <ImageView
        android:id="@+id/img2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:src="@drawable/ic_launcher" />

    <ImageView
        android:id="@+id/img3"
        android:onClick="AA"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:background="@android:color/black"
        android:src="@drawable/ic_launcher" />

    <ImageView
        android:id="@+id/img4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:src="@drawable/ic_launcher" />

    <ImageView
        android:id="@+id/img5"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:src="@drawable/ic_launcher" />

    <ImageView
        android:id="@+id/img6"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:src="@drawable/ic_launcher" />

</LinearLayout>
    {% endhighlight %}

其他辅助代码，如动画和act等内容

    {% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<scale
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:fromXScale="1.0"
    android:fromYScale="1.0"
    android:toXScale="3.0"
    android:toYScale="3.0"
    android:duration="2000"
       android:pivotX="50%"
       android:pivotY="50%"
    >
</scale>

     {% endhighlight %}

    {% highlight java %}
import android.os.Bundle;
import android.app.Activity;
import android.view.Menu;
import android.view.View;
import android.view.animation.Animation;
import android.view.animation.AnimationUtils;
import android.widget.ImageView;
import android.widget.Toast;

public class MainActivity extends Activity {

    //只对第三个小人做放大动作
    ImageView image3 =null;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        image3 = (ImageView) findViewById(R.id.img3);
    }


    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate the menu; this adds items to the action bar if it is present.
        getMenuInflater().inflate(R.menu.main, menu);
        return true;
    }

    public void AA(View view) {
        Toast.makeText(this, "aa", Toast.LENGTH_SHORT).show();
        Animation an = AnimationUtils.loadAnimation(getApplicationContext(), R.anim.anims);
        image3.startAnimation(an);
    }

}

     {% endhighlight %}