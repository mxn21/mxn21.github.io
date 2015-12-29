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
        private Rect mRect;
        private GradientDrawable mDrawable;

        public SampleView(Context context) {
            super(context);
            setFocusable(true);

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
        
第二个参数是一个color数组，相当于startColor，endColor，centerColor，其中centerColor可以省略,但是至少要设置两个颜色。
mDrawable.setGradientType可以设置三种type同xml一样，分别是GradientDrawable.LINEAR_GRADIENT，GradientDrawable
.RADIAL_GRADIENT，GradientDrawable.SWEEP_GRADIENT。


### Shader类的子类

Shader类的子类创建允许使用多种固体颜色填充绘图对象的Paint，功能不只是实现渐变填充。有三Shader是用来做渐变的：
LinearGradient、RadialGradient和 SweepGradient. 看名字就知道这三种和上面的GradientDrawable的三种type是对应的。
只不过是用Shader实现了。

要在绘图的时候使用一个Shader，可以使用setShader方法将其应用到一个Paint中，如下面的代码所示：

      {% highlight java  %}
    Paint shaderPaint = new Paint();  
    shaderPaint.setShader(myLinearGradient); 
        {% endhighlight %}
 
使用这个Paint所绘制的任何东西都将使用你指定的Shader进行填充，而不是使用Paint本身的颜色进行填充。        
下面使用LinearGradient实现渐变，对于RadialGradient和 SweepGradient使用很类似。
      
        {% highlight java  %}    
        public class TestActivity extends Activity {
        
            @Override
            protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(new SampleView(this));
            }
        
            private static class SampleView extends View {
                private Paint mPaint;
                private Rect mRect;
                LinearGradient lg1 ;
                LinearGradient lg2 ;
                LinearGradient lg3 ;
        
                public SampleView(Context context) {
                    super(context);
                    setFocusable(true);
        
                    mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
                    mRect = new Rect(0, 0, 300, 300);
        
                     lg1 = new LinearGradient(0,0,150,150, Color.TRANSPARENT,Color.BLACK,
                            Shader.TileMode.MIRROR);
                     lg2 = new LinearGradient(0,0,150,150, Color.TRANSPARENT,Color.BLACK,
                            Shader.TileMode.CLAMP);
                     lg3 = new LinearGradient(0,0,150,150, Color.TRANSPARENT,Color.BLACK,
                            Shader.TileMode.REPEAT);
        
                }
        
                @Override protected void onDraw(Canvas canvas) {
                    canvas.save();
                    canvas.translate(10, 10);
                    mPaint.setShader(lg1) ;
                    canvas.drawRect(mRect,mPaint);
                    canvas.restore();
        
                    canvas.save();
                    canvas.translate(10 + mRect.width() + 10, 10);
                    mPaint.setShader(lg2) ;
                    canvas.drawRect(mRect,mPaint);
                    canvas.restore();
        
                    canvas.save();
                    canvas.translate(10,10 + mRect.height() + 10);
                    mPaint.setShader(lg3) ;
                    canvas.drawRect(mRect,mPaint);
                    canvas.restore();
                }
            }
        }
        {% endhighlight %}      
        
效果如下：
     
上面的例子使用了三种Shader TileModes，如果Shader画刷所定义的区域比要填充的区域小，那么TileMode将会决定如何处理剩余的区域：

MIRROR  在水平和垂直方向上拉伸Shader图像，这样每一个图像就都能与上一个缝合了。

CLAMP  使用Shader的边界颜色来填充剩余的空间。

REPEAT  在水平和垂直方向上重复Shader图像，但不拉伸它。  

LinearGradient有两种方式实例化：
      
               {% highlight java  %} 
LinearGradient(float x0, float y0, float x1, float y1, int[] colors, float[] positions, Shader.TileMode tile)
LinearGradient(float x0, float y0, float x1, float y1, int color0, int color1, Shader.TileMode tile)
               {% endhighlight %}  
                
他们的不同之处为参数中第一种方法可以用颜色数组，和位置来实现更细腻的过渡效果，
比如颜色采样int[] colors数组中存放20种颜色，则渐变将会逐一处理。而第二种方法参数仅为起初颜色color0和最终颜色color1。               
        
        
### 自定义渐变

可以利用工具类重新计算LinearGradient的颜色参数，从而实现更柔和的渐变，仍然使用最开始的LinearLayout布局测试：

       {% highlight java  %} 
     
     public class TestActivity extends Activity  {
     
         @Override
         protected void onCreate(Bundle savedInstanceState) {
             super.onCreate(savedInstanceState);
             setContentView(R.layout.activity_test);
     
             if(Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.JELLY_BEAN) {
                 View bottom = findViewById(R.id.layout_bottom);
                 bottom.setBackground(
                         ScrimUtil.makeCubicGradientScrimDrawable(
                                 0xaa000000, //颜色
                                 8, //渐变层数
                                 Gravity.BOTTOM)); //起始方向
             }
         }
     
     }
       {% endhighlight %}    

ScrimUtil代码如下：

       {% highlight java  %} 
public class ScrimUtil {

    private ScrimUtil() {
    }

    /**
     * Creates an approximated cubic gradient using a multi-stop linear gradient. See
     * <a href="https://plus.google.com/+RomanNurik/posts/2QvHVFWrHZf">this post</a> for more
     * details.
     */
    public static Drawable makeCubicGradientScrimDrawable(int baseColor, int numStops, int gravity) {
        numStops = Math.max(numStops, 2);

        PaintDrawable paintDrawable = new PaintDrawable();
        paintDrawable.setShape(new RectShape());

        final int[] stopColors = new int[numStops];

        int red = Color.red(baseColor);
        int green = Color.green(baseColor);
        int blue = Color.blue(baseColor);
        int alpha = Color.alpha(baseColor);

        for (int i = 0; i < numStops; i++) {
            float x = i * 1f / (numStops - 1);
            float opacity = constrain(0, 1, (float) Math.pow(x, 3));
            stopColors[i] = Color.argb((int) (alpha * opacity), red, green, blue);
        }

        final float x0, x1, y0, y1;
        switch (gravity & Gravity.HORIZONTAL_GRAVITY_MASK) {
            case Gravity.LEFT:  x0 = 1; x1 = 0; break;
            case Gravity.RIGHT: x0 = 0; x1 = 1; break;
            default:            x0 = 0; x1 = 0; break;
        }
        switch (gravity & Gravity.VERTICAL_GRAVITY_MASK) {
            case Gravity.TOP:    y0 = 1; y1 = 0; break;
            case Gravity.BOTTOM: y0 = 0; y1 = 1; break;
            default:             y0 = 0; y1 = 0; break;
        }

        paintDrawable.setShaderFactory(new ShapeDrawable.ShaderFactory() {
            @Override
            public Shader resize(int width, int height) {
                LinearGradient linearGradient = new LinearGradient(
                        width * x0,
                        height * y0,
                        width * x1,
                        height * y1,
                        stopColors, null,
                        Shader.TileMode.CLAMP);
                return linearGradient;
            }
        });

        return paintDrawable;
    }

    private  static float constrain(float min, float max, float v) {
        return Math.max(min, Math.min(max, v));
    }
}
       {% endhighlight %}  
       
最终效果：


