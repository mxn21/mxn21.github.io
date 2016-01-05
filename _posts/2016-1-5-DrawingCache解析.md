---
layout: post
author: mxn
titile: DrawingCache解析
category: 技术博文
tag: android
---

在Android的View的设计中有一个cache的概念，这个cache就是一个bitmap对象。利用这个功能可以对整个屏幕视图进行截屏并生成Bitmap，也可以
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
    
