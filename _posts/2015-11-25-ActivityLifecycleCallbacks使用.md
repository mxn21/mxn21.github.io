---
layout: post
author: mxn
titile: ActivityLifecycleCallbacks使用
category: 技术博文
tag: android
---

Application通过ActivityLifecycleCallbacks使用接口提供了一套回调方法，用于让开发者对Activity的生命周期事件进行集中处理。
ActivityLifecycleCallbacks接口回调可以简化监测Activity的生命周期事件，在一个类中作统一处理。
ActivityLifecycleCallbacks使用要求API 14+ （Android 4.0+）。

### 使用方法

重写Application的onCreate()方法，或在Application的无参构造方法内，调用Application.registerActivityLifecycleCallbacks()方法
，并实现ActivityLifecycleCallbacks接口。

<!-- more -->

    {% highlight java %}
public class BaseApplication extends Application {

    public void onCreate() {
        super.onCreate();
        this.registerActivityLifecycleCallbacks(new ActivityLifecycleCallbacks() {

            @Override
            public void onActivityStopped(Activity activity) {
                Log.e("====", activity+"onActivityStopped");
            }

            @Override
            public void onActivityStarted(Activity activity) {
                Log.e("====", activity+"onActivityStarted");
            }

            @Override
            public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
                Log.e("====", activity + "onActivitySaveInstanceState");
            }

            @Override
            public void onActivityResumed(Activity activity) {
                Log.e("====", activity + "onActivityResumed");
            }

            @Override
            public void onActivityPaused(Activity activity) {
                Log.e("====", activity + "onActivityPaused");
            }

            @Override
            public void onActivityDestroyed(Activity activity) {
                Log.e("====", activity + "onActivityDestroyed");
            }

            @Override
            public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
                Log.e("====", activity + "onActivityCreated");
            }
        });
    };
}

    {% endhighlight %}
