---
layout: post
author: mxn
titile: DrawingCache解析
category: 技术博文
tag: android
---

android为了提高滚动等各方面的绘制速度，可以为每一个view建立一个缓存，使用 View.buildDrawingCache为自己的view建立相应的缓存，
这个cache就是一个bitmap对象。利用这个功能可以对整个屏幕视图进行截屏并生成Bitmap，也可以
获得指定的view的Bitmap对象。在有的时候还会影响性能，例如如果自己实现一个Gallery效果,可能就会使用到view缓存。animateCache和scrollingCache
用于动画和滚动的缓存，使用不当也会造成性能下降。

要获得一个view的bitmap对象涉及到三个方法：setDrawingCacheEnabled、buildDrawingCache和getDrawingCache。所有的View都有这三种方法。
大部分view如果没有设置setDrawingCacheEnabled(true);来启用View的DrawingCache功能的话，那默认是不启用。

启用DrawingCache的话，使用getDrawingCache方法时，会先自动去调用buildDrawingCache方法建立DrawingCache，再将结果返回；
不启用DrawingCache的話，使用getDrawingCache方法時，会返回上一次使用buildDrawingCache方法所产生的结果。
如果在此之前都沒有使用过buildDrawingCache來建立DrawingCache的话，那么getDrawingCache就会返回null。
如果一开始没有启用DrawingCache，也是可以事先使用buildDrawingCache来建立DrawingCache，避免getDrawingCache返回null。

getDrawingCache源码如下：

    {% highlight java  %} 
  public Bitmap getDrawingCache() {
        return getDrawingCache(false);
    }
  
  public Bitmap getDrawingCache(boolean autoScale) {
          if ((mViewFlags & WILL_NOT_CACHE_DRAWING) == WILL_NOT_CACHE_DRAWING) {
              return null;
          }
          if ((mViewFlags & DRAWING_CACHE_ENABLED) == DRAWING_CACHE_ENABLED) {
              buildDrawingCache(autoScale);
          }
          return autoScale ? mDrawingCache : mUnscaledDrawingCache;
      }
    {% endhighlight %} 
    
可以看出getDrawingCache在设置了DrawingCache的情况下自动调用buildDrawingCache。

照刚才所说的，那么要获得最新的DrawingCache有两种方式：

方式一：

    {% highlight java  %} 
view.setDrawingCacheEnabled(true);
Bitmap drawingCache = view.getDrawingCache();
    {% endhighlight %} 
    
方式二：
    
    {% highlight java  %}     
view.buildDrawingCache();
Bitmap drawingCache = view.getDrawingCache();
    {% endhighlight %} 
    
在调用setDrawingCacheEnabled(true);以后就不要再调用buildDrawingCache方法了，以下写法应该避免，会两次建立DrawingCache：

    {% highlight java  %}  
view.setDrawingCacheEnabled(true);
view.buildDrawingCache();
Bitmap drawingCache = view.getDrawingCache();
    {% endhighlight %} 
    
buildDrawingCache建立drawingCache的同时，会将上次的DrawingCache回收掉，在源码中buildDrawingCache
会调用destroyDrawingCache方法对之前的DrawingCache回收，源码如下：

    {% highlight java  %}  
/**
     * <p>Frees the resources used by the drawing cache. If you call
     * {@link #buildDrawingCache()} manually without calling
     * {@link #setDrawingCacheEnabled(boolean) setDrawingCacheEnabled(true)}, you
     * should cleanup the cache with this method afterwards.</p>
     *
     * @see #setDrawingCacheEnabled(boolean)
     * @see #buildDrawingCache()
     * @see #getDrawingCache()
     */
    public void destroyDrawingCache() {
        if (mDrawingCache != null) {
            mDrawingCache.recycle();
            mDrawingCache = null;
        }
        if (mUnscaledDrawingCache != null) {
            mUnscaledDrawingCache.recycle();
            mUnscaledDrawingCache = null;
        }
    }
    
     {% endhighlight %} 
     
因此不必在buildDrawingCache方法之前，或者DrawingCache启用状态下调用getDrawingCache方法之前，自己手动调用destroyDrawingCache。
会导致RumtimeException:java.lang.RuntimeException: Canvas: trying to use a recycled bitmap android.graphics.Bitmap@4b8eb8

下面的写法是错误写法：

    {% highlight java  %}  
    if(view.getDrawingCache() != null){
    	view.getDrawingCache().recycle();;
    }
    view.buildDrawingCache();
    Bitmap drawingCache = view.getDrawingCache();
    {% endhighlight %} 
    
    
### 图片质量控制

对于Bitmap对象可以有多种格式，如：

Bitmap.Config.ARGB_8888;

Bitmap.Config.ARGB_4444;

Bitmap.Config.ARGB_8888;
                   
Bitmap.Config.ARGB_8888;
                   
Bitmap.Config.RGB_565；

默认的格式是Bitmap.Config.ARGB_8888，但大多数嵌入式设备使用的显示格式都是Bitmap.Config.RGB_565. RGB_565并没有alpha值，
所以绘制的时候不需要计算alpha合成，速度快些。其次，RGB_565可以直接使用优化了的memcopy函数，效率相对高出许多。

可以用以下方法查看bitmap格式：

    {% highlight java  %}  
    final Bitmap cache = mContent.getDrawingCache();
     if (cache != null) {
            Config cfg = cache.getConfig();
            Log.d(TAG, "----------------------- cache.getConfig() = " + cfg);
      }
    {% endhighlight %} 

随着Android API越来越高，DrawingCache的质量也越来越，在大部分的情况下都是使用体积最大且运算速度最慢的ARGB_8888，
过去View所提供的setDrawingCacheQuality方法已經沒有实际作用了，不管设定哪种质量，都还是会使用ARGB_8888。

### getDrawingCache返回空

一种可能是view没有初始化完成，onCreate中view还没有初始化自己的宽高，所以getDrawingCache();返回空。可以参考[viewTreeObserver解析](http://souly.cn/技术博文/2015/11/16/viewTreeObserver解析/)
这篇来获取view宽高。

下面给出两种方法：

    {% highlight java  %}  
  public static Bitmap convertViewToBitmap(View view){
        view.measure(View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED), View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED));
        view.layout(0, 0, view.getMeasuredWidth(), view.getMeasuredHeight());
        view.buildDrawingCache();
        Bitmap bitmap = view.getDrawingCache();
        return bitmap;
    }
    {% endhighlight %} 
    
第二种方法利用ViewTreeObserver

    {% highlight java  %}  
        ViewTreeObserver vto = view.getViewTreeObserver();
        vto.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
            @Override
            public void onGlobalLayout() {
                view.getViewTreeObserver().removeGlobalOnLayoutListener(this);
                Bitmap cac = view.getDrawingCache();
                if(cac != null) {
                    Bitmap.Config cfg = cac.getConfig();
                    Log.e("====", "not null"+cfg);
                    mImageView2.setImageBitmap(cac);
                } else {
                    Log.e("====", "null");
                }
            }
        });

   {% endhighlight %} 

如果很确定View已经有过measure和layout且也调用buildDrawingCache(无论自动或者手动)方法了，但是getDrawingCache还是返回null，
那就是因为要绘制的DrawingCache太大了，超过Android系统设定的drawingCacheSize，这时，就只能放弃使用DrawingCache了。

Android系统设定的DrawingCache大小上限，在不同的裝置上有不同的设定，甚至有可能差了好几倍，如果要查看数值的話可以使用以下方式來取得drawingCacheSize：

    {% highlight java  %}  
ViewConfiguration.get(context).getScaledMaximumDrawingCacheSize();
   {% endhighlight %} 
   
   
### getDrawingCache的替代方法

如果不用getDrawingCache想自己建立出Bitmap也是可以的，代码如下：

    {% highlight java  %}
     
    public Bitmap getMagicDrawingCache(View view) {
        Bitmap bitmap = (Bitmap) view.getTag(R.id.cacheBitmapKey);
        Boolean dirty = (Boolean) view.getTag(R.id.cacheBitmapDirtyKey);
        int viewWidth = view.getWidth();
        int viewHeight = view.getHeight();
        if (bitmap == null || bitmap.getWidth() != viewWidth || bitmap.getHeight() != viewHeight) {
            if (bitmap != null && !bitmap.isRecycled()) {
                bitmap.recycle();
            }
            bitmap = Bitmap.createBitmap(viewWidth, viewHeight, bitmap_quality);
            view.setTag(R.id.cacheBitmapKey, bitmap);
            dirty = true;
        }
        if (dirty == true || !quick_cache) {
            bitmap.eraseColor(getResources().getColor(android.R.color.transparent));
            Canvas canvas = new Canvas(bitmap);
            view.draw(canvas);
            view.setTag(R.id.cacheBitmapDirtyKey, false);
        }
        return bitmap;
    }
    {% endhighlight %} 
    
如果要加入View不在Activity或是Fragment的RootView中的判断的话，代码如下：

    {% highlight java  %}
public Bitmap getMagicDrawingCache2(View view) {
        Bitmap bitmap = (Bitmap) view.getTag(R.id.cacheBitmapKey);
        Boolean dirty = (Boolean) view.getTag(R.id.cacheBitmapDirtyKey);
        if (view.getWidth() + view.getHeight() == 0) {
            view.measure(View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED), View.MeasureSpec.makeMeasureSpec(0, View.MeasureSpec.UNSPECIFIED));
            view.layout(0, 0, view.getMeasuredWidth(), view.getMeasuredHeight());
        }
        int viewWidth = view.getWidth();
        int viewHeight = view.getHeight();
        if (bitmap == null || bitmap.getWidth() != viewWidth || bitmap.getHeight() != viewHeight) {
            if (bitmap != null && !bitmap.isRecycled()) {
                bitmap.recycle();
            }
            bitmap = Bitmap.createBitmap(viewWidth, viewHeight, bitmap_quality);
            view.setTag(R.id.cacheBitmapKey, bitmap);
            dirty = true;
        }
        if (dirty == true || !quick_cache) {
            bitmap.eraseColor(getResources().getColor(android.R.color.transparent));
            Canvas canvas = new Canvas(bitmap);
            view.draw(canvas);
            view.setTag(R.id.cacheBitmapDirtyKey, false);
        }
        return bitmap;
    }
        {% endhighlight %} 
        
其中，cacheBitmapKey和cacheBitmapDirtyKey是不同的整数，分别用来指定View的Tag ID。cacheBitmapKey的位置会存放使用
这个方法建立出来的DrawingCache；cacheBitmapDirtyKey的位置会存放这个View的DrawingCache是否已经是脏数据(dirty)而需要使用
View的draw方法重新绘制。DrawingCache所用的Bitmap只在没有Bitmap或是Bitmap的大小和View的大小不合的时候才重新建立，
在建立新的Bitmap前会先將先前的Bitmap进行recycle，新的Bitmap的参考会再被存入至View的Tag中。他们的定义如下：

    {% highlight xml  %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item type="id" name="cacheBitmapKey"></item>
    <item type="id" name="cacheBitmapDirtyKey"></item>
</resources>
    {% endhighlight %} 
    
另外需要设置bitmap_quality和quick_cache：
    
      {% highlight java  %}  
    Bitmap.Config bitmap_quality = Bitmap.Config.ARGB_8888 ;
    boolean quick_cache = false ;
        {% endhighlight %} 
        
quick_cache若设置为false，则不论DrawingCache是否dirty，都进行重绘，只有在View常常变化的时候才需要这样做。
bitmap_quality可以设置为Bitmap.Config.RGB_565或是Bitmap.Config.ARGB_8888，Bitmap.Config.ARGB_4444已经随
着Android API升级家而慢慢被禁用了。


### scrollingCache和animateCache

scrollingCache是listview这种滚动布局的一个属性，animateCache是viewgroup的一个属性。他们的作用都是控制DrawingCache。
他们都可以在xml布局中控制，也可以用代码调用：

      {% highlight java  %}  
 mylayout.setAnimationCacheEnabled(false);
      {% endhighlight %} 
      
setAnimationCacheEnabled源码如下：

      {% highlight java  %} 
 /**
     * Enables or disables the children's drawing cache during a layout animation.
     * By default, the drawing cache is enabled but this will prevent nested
     * layout animations from working. To nest animations, you must disable the
     * cache.
     *
     * @param enabled true to enable the animation cache, false otherwise
     *
     * @see #isAnimationCacheEnabled()
     * @see View#setDrawingCacheEnabled(boolean)
     */
    public void setAnimationCacheEnabled(boolean enabled) {
        setBooleanFlag(FLAG_ANIMATION_CACHE, enabled);
    }
    
   {% endhighlight %} 
   
方法的注释说他的功能是在执行一个Layout动画时开启或关闭子控件的绘制缓存。默认情况下，绘制缓存是开启的，但是这将阻止嵌套Layout动画的正常执行。
对于嵌套动画，你必须禁用这个缓存。这个属性如果设置true后，在动画绘制过程中会为每一个子布局设置cache，这会提高显示效果，
但是需要消耗更多内存和更长的初始化时间。这个属性默认是true。

为什么设置了缓存，动画会更加平滑，是因为避免了在每一帧的重绘。设置了缓存的动画还可以被硬件加速，因为在硬件层，渲染系统
可以把bitmap交给GPU处理，并对其进行快速的矩阵操作（如改变透明度，平移、旋转）。而不使用缓存的情况下，则是在每一帧进行
重绘，即调用onDraw()方法。

scrollingCache属性和animateCache相似,源码如下：

      {% highlight java  %} 
/**
     * Enables or disables the children's drawing cache during a scroll.
     * By default, the drawing cache is enabled but this will use more memory.
     *
     * When the scrolling cache is enabled, the caches are kept after the
     * first scrolling. You can manually clear the cache by calling
     * {@link android.view.ViewGroup#setChildrenDrawingCacheEnabled(boolean)}.
     *
     * @param enabled true to enable the scroll cache, false otherwise
     *
     * @see #isScrollingCacheEnabled()
     * @see View#setDrawingCacheEnabled(boolean)
     */
    public void setScrollingCacheEnabled(boolean enabled) {
        if (mScrollingCacheEnabled && !enabled) {
            clearScrollingCache();
        }
        mScrollingCacheEnabled = enabled;
    }
   {% endhighlight %} 
   
对于listview当滚动的时候，实际上是可见的item布局的执行了动画，使用缓存可以加速动画。但是他的缺点就是它消耗的内存。
所以可以手动设置关闭，对于流畅性目前并没有发现有什么影响。

优化后的listview：

      {% highlight xml  %} 
<ListView
    android:id="@android:id/list"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:divider="@color/list_background_color"
    android:dividerHeight="0dp"
    android:listSelector="#00000000"
    android:smoothScrollbar="true"
    android:scrollingCache="false"
    android:animationCache="false" />

   {% endhighlight %}
    
 
下面写一个demo验证chche对内存的影响
首先关闭硬件加速

      {% highlight xml  %} 
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.mxn.soul.demo" >

    <application
        android:allowBackup="true"
        android:icon="@mipmap/icon"
        android:label="@string/app_name"
        android:name=".BaseApplication"
        android:theme="@style/AppTheme" >
        <activity
            android:name=".Test5Activity"
            android:hardwareAccelerated="false"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
        {% endhighlight %}
      
      
在一个LinearLayout中放16个imageview，让这个LinearLayout执行缩小动画，imageview执行旋转动画

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img142.png)

      {% highlight java  %} 
public class Test5Activity extends Activity  {

    private ImageView[] mImageViews = new ImageView[16];
    private int[] mImageViewIDs = {R.id.img1,R.id.img2,R.id.img3,R.id.img4,R.id.img5,R.id.img6,R
            .id.img7,R.id.img8,R.id.img9,R.id.img10,R.id.img11,R.id.img12,R.id.img13,R.id.img14,R
            .id.img15,R.id.img16} ;
    private LinearLayout mylayout ;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test5);
        mylayout = (LinearLayout) findViewById(R.id.mylayout);
        for(int n = 0 ; n < mImageViews.length ; n ++ ){
            mImageViews[n] = (ImageView) findViewById(mImageViewIDs[n]);
        }

        mylayout.setAnimationCacheEnabled(true);
        mylayout.setOnClickListener(new View.OnClickListener() {
            public void onClick(View arg0) {
                doAnimation() ;
            }
        });
    }


    public void doAnimation() {
        AnimationSet animationSet=new AnimationSet(true);
        ScaleAnimation scaleAnimation=new ScaleAnimation(
                1, 0.1f, 1, 0.1f,
                Animation.RELATIVE_TO_SELF, 0.5f,
                Animation.RELATIVE_TO_SELF, 0.5f);
        scaleAnimation.setDuration(2000);
        animationSet.addAnimation(scaleAnimation);
        mylayout.startAnimation(scaleAnimation) ;

        RotateAnimation rotateAnimation =new RotateAnimation(0f,360, Animation.RELATIVE_TO_SELF,
                0.5f,Animation.RELATIVE_TO_SELF,0.5f);
        rotateAnimation.setDuration(2000);
        animationSet.addAnimation(rotateAnimation);

        for(int n = 0 ; n < mImageViews.length ; n ++ ){
            mImageViews[n].startAnimation(rotateAnimation) ;
        }

    }
}
   {% endhighlight %}
   
最终效果如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img143.gif)

观察内存使用情况，设置mylayout.setAnimationCacheEnabled(false);时如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img144.png)

可以看出不管动画如何变化，内存没有变化。

设置mylayout.setAnimationCacheEnabled(true);时如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img145.png)

可以看出不管动画变化是，内存在不断增加，之后被回收，因为缓存不断地产生了新的bitmap。对于动画地流畅性来说几乎
看出有什么不同。

为了更清楚的观察设置了缓存后onDraw方法的调用情况，我们用自定义的view代替ImageView.

      {% highlight java  %} 
public class MyImageView extends ImageView {
    static int count  = 0 ;
    public MyImageView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    public MyImageView(Context context) {
        super(context);
    }
    @Override
    protected void onMeasure(int widthMeasureSpec,int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        if (BuildConfig.DEBUG) Log.d("===MyImageView","onMeasure 我被调用了"+System.currentTimeMillis());
    }
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        count++ ;
        if (BuildConfig.DEBUG) Log.d("===MyImageView", "onDraw 我被调用了"+System.currentTimeMillis()
                +"==="+count);
    }
}
   {% endhighlight %}
   
使用一个静态变量count记录onDraw调用的次数。

