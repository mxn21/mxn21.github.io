---
layout: post
author: mxn
titile: DrawingCache解析
category: 技术博文
tag: android
---

android为了提高滚动等各方面的绘制速度，可以为每一个view建立一个缓存，使用 View.buildDrawingCache为自己的view建立相应的缓存，
这个cache就是一个bitmap对象。利用这个功能可以对整个屏幕视图进行截屏并生成Bitmap，也可以
获得指定的view的Bitmap对象。在有的时候还会影响性能，例如如果自己实现一个Gallery效果,可能就会使用到view缓存。

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
