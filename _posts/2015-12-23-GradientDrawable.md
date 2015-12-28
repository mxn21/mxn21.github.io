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
   
效果图如下：



上面的例子中我们在gradient标签中设置了startColor，endColor，angle用来表示开始结束的颜色和变化方向。
gradient标签的所有属性说明如下：
     
android:angle：（Integer） 渐变的角度，线性渐变时才有效，必须是45的倍数，0表示从左到右，90表示从下到上

android:centerX：（Float）渐变中心的相对X坐标，放射渐变时才有效，在0.0到1.0之间，默认为0.5，表示在正中间

android:centerY：（Float）渐变中心的相对X坐标，放射渐变时才有效，在0.0到1.0之间，默认为0.5，表示在正中间

android:centerColor ：（Color）中间点的色值

android:endColor ： （Color）结束的色值

android:startColor：（Color）开始的色值。

android:gradientRadius：（Float）渐变的半径，只有在android:type="radial"的时候有效。

android:type ：有三种类型
              "linear" 线性渐变，
              "radial" 放射渐变，设置该项时，android:gradientRadius也必须设置
              "sweep"  扫描性渐变
 
android:useLevel ： 如果为true，将被当成LevelListDrawable使用。
    
    
除了用xml设置，还可以在编码中设置，<gradient>标签对应的类是GradientDrawable，GradientDrawable是Drawable的子类。
代码如下：

      {% highlight java  %}
public class TestActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(new SampleView(this));
    }

    private static class SampleView extends View {
        private Path mPath;
        private Paint mPaint;
        private Rect mRect;
        private GradientDrawable mDrawable;

        public SampleView(Context context) {
            super(context);
            setFocusable(true);

            mPath = new Path();
            mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
            mRect = new Rect(0, 0, 300, 300);

            mDrawable = new GradientDrawable(GradientDrawable.Orientation.TL_BR,
                    new int[] { 0xaa000000,
                            0xFFFFFFFF });
            mDrawable.setShape(GradientDrawable.RECTANGLE);
            mDrawable.setGradientRadius((float)(Math.sqrt(2) * 60));
        }

        static void setCornerRadii(GradientDrawable drawable, float r0,
                                   float r1, float r2, float r3) {
            drawable.setCornerRadii(new float[] { r0, r0, r1, r1,
                    r2, r2, r3, r3 });
        }

        @Override protected void onDraw(Canvas canvas) {

            mDrawable.setBounds(mRect);

            float r = 16;

            canvas.save();
            canvas.translate(10, 10);
            mDrawable.setGradientType(GradientDrawable.LINEAR_GRADIENT);
            setCornerRadii(mDrawable, r, r, 0, 0);
            mDrawable.draw(canvas);
            canvas.restore();

            canvas.save();
            canvas.translate(10 + mRect.width() + 10, 10);
            mDrawable.setGradientType(GradientDrawable.RADIAL_GRADIENT);
            setCornerRadii(mDrawable, 0, 0, r, r);
            mDrawable.draw(canvas);
            canvas.restore();

            canvas.translate(0, mRect.height() + 10);

            canvas.save();
            canvas.translate(10, 10);
            mDrawable.setGradientType(GradientDrawable.SWEEP_GRADIENT);
            setCornerRadii(mDrawable, 0, r, r, 0);
            mDrawable.draw(canvas);
            canvas.restore();

        }
    }
}

        {% endhighlight %}

效果如下图：


可以看到，代码设置和xml设置大同小异，注意实例化的操作：

      {% highlight java  %}
 public GradientDrawable(Orientation orientation, int[] colors) {
        this(new GradientState(orientation, colors));
    }
        {% endhighlight %}

第一个参数是一个枚举，表示渐变方向，这个用来相当于xml里面的angle，只不过angle是用45的倍数表示方向，而枚举看上去更清楚了。

      {% highlight java  %}
  public enum Orientation {
        /** draw the gradient from the top to the bottom */
        TOP_BOTTOM,
        /** draw the gradient from the top-right to the bottom-left */
        TR_BL,
        /** draw the gradient from the right to the left */
        RIGHT_LEFT,
        /** draw the gradient from the bottom-right to the top-left */
        BR_TL,
        /** draw the gradient from the bottom to the top */
        BOTTOM_TOP,
        /** draw the gradient from the bottom-left to the top-right */
        BL_TR,
        /** draw the gradient from the left to the right */
        LEFT_RIGHT,
        /** draw the gradient from the top-left to the bottom-right */
        TL_BR,
    }
        {% endhighlight %}
        
第二个参数是一个

     
     
     
