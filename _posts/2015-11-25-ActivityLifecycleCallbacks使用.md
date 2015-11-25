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

为了测试，再新建两个activity：MainActivity和SecondActivity，点击MainActivity里面的按钮可以跳转到SecondActivity。
MainActivity代码如下：

    {% highlight java %}
    public class MainActivity extends ActionBarActivity {
        private Button btn = null;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            btn = (Button) findViewById(R.id.btn);
            btn.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    Intent i = new Intent(MainActivity.this,SecondActivity.class);
                    startActivity(i);
                }
            });
            Log.e("====", "MainActivity  onCreate");
        }
    
    
        @Override
        protected void onResume() {
            super.onResume();
            Log.e("====", "MainActivity  onResume");
        }
    
        @Override
        protected void onPause() {
            super.onPause();
            Log.e("====", "MainActivity  onPause");
        }
    
        @Override
        protected void onStart() {
            super.onStart();
            Log.e("====", "MainActivity  onStart");
        }
    }
    {% endhighlight %}
    
SecondActivity代码类似，不再贴出来。运行app，输出如下

    {% highlight c %}
    11-25 16:45:24.043 2829-2829/com.mxn.soul.demo E/====: com.mxn.soul.demo.MainActivity@2300e06conActivityCreated
    11-25 16:45:24.179 2829-2829/com.mxn.soul.demo E/====: MainActivity  onCreate
    11-25 16:45:24.238 2829-2829/com.mxn.soul.demo E/====: com.mxn.soul.demo.MainActivity@2300e06conActivityStarted
    11-25 16:45:24.240 2829-2829/com.mxn.soul.demo E/====: MainActivity  onStart
    11-25 16:45:24.242 2829-2829/com.mxn.soul.demo E/====: com.mxn.soul.demo.MainActivity@2300e06conActivityResumed
    11-25 16:45:24.242 2829-2829/com.mxn.soul.demo E/====: MainActivity  onResume
    11-25 16:45:36.768 2829-2829/com.mxn.soul.demo E/====: com.mxn.soul.demo.MainActivity@2300e06conActivityPaused
    11-25 16:45:36.768 2829-2829/com.mxn.soul.demo E/====: MainActivity  onPause
    11-25 16:45:36.820 2829-2829/com.mxn.soul.demo E/====: com.mxn.soul.demo.SecondActivity@13adc62aonActivityCreated
    11-25 16:45:36.856 2829-2829/com.mxn.soul.demo E/====: SecondActivity  onCreate
    11-25 16:45:36.858 2829-2829/com.mxn.soul.demo E/====: com.mxn.soul.demo.SecondActivity@13adc62aonActivityStarted
    11-25 16:45:36.858 2829-2829/com.mxn.soul.demo E/====: SecondActivity onStart
    11-25 16:45:36.859 2829-2829/com.mxn.soul.demo E/====: com.mxn.soul.demo.SecondActivity@13adc62aonActivityResumed
    11-25 16:45:36.859 2829-2829/com.mxn.soul.demo E/====: SecondActivity onResume
    11-25 16:45:37.321 2829-2829/com.mxn.soul.demo E/====: com.mxn.soul.demo.MainActivity@2300e06conActivitySaveInstanceState
    11-25 16:45:37.321 2829-2829/com.mxn.soul.demo E/====: com.mxn.soul.demo.MainActivity@2300e06conActivityStopped
    {% endhighlight %}
    
可以看到ActivityLifecycleCallbacks的回调在activity生命周期方法之前，这是因为我的log信息写在super之后：

    {% highlight c %}
     @Override
    protected void onResume() {
        super.onResume();
         Log.e("====", "MainActivity  onResume");
    }
    
    @Override
    protected void onResume() {
        Log.e("====", "MainActivity  onResume");
        super.onResume();
    }