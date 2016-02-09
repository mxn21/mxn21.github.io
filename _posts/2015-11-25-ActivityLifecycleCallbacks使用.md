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

    {% highlight java %}
     @Override
    protected void onResume() {
        super.onResume();
         Log.e("====", "MainActivity  onResume");
    }
    {% endhighlight %}
    
如果把log写在super之前，那么结果就是相反的了。
    
    {% highlight java %}
    @Override
    protected void onResume() {
        Log.e("====", "MainActivity  onResume");
        super.onResume();
    }
    {% endhighlight %}
    
输出如下：

    {% highlight c %}
    11-25 17:28:15.652 27469-27469/? E/====: MainActivity  onResume
    11-25 17:28:15.653 27469-27469/? E/====: com.mxn.soul.demo.MainActivity@2300e06conActivityResumed
    {% endhighlight %}
    
### 用ActivityLifecycleCallbacks管理Activity堆栈

可以用ActivityLifecycleCallbacks和LinkedList来管理所有的Activity，可以实现完全退出一个应用。

    {% highlight java %}
public class BaseApplication extends Application  implements Application.ActivityLifecycleCallbacks {
    private LinkedList<ActivityInfo> mExistedActivitys = new LinkedList<>();
    private static BaseApplication mInstance;
    public static  synchronized BaseApplication getInstance() {
        return mInstance;
    }
    public void onCreate() {
        super.onCreate();
        registerActivityLifecycleCallbacks(this);
         mInstance = this;
    }
    @Override
    public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
        if (null != mExistedActivitys && null != activity) {
            // 把新的 activity 添加到最前面，和系统的 activity 堆栈保持一致
            mExistedActivitys.offerFirst(new ActivityInfo(activity,ActivityInfo.STATE_CREATE));
        }
    }
    @Override
    public void onActivityStarted(Activity activity) {
    }
    @Override
    public void onActivityResumed(Activity activity) {
    }
    @Override
    public void onActivityPaused(Activity activity) {
    }
    @Override
    public void onActivityStopped(Activity activity) {
    }
    @Override
    public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
    }
    @Override
    public void onActivityDestroyed(Activity activity) {
        if (null != mExistedActivitys && null != activity) {
            ActivityInfo info = findActivityInfo(activity);
            if (null != info) {
                mExistedActivitys.remove(info);
            }
        }
    }
    class ActivityInfo {
        private final static int STATE_NONE = 0;
        private final static int STATE_CREATE = 1;
        Activity mActivity;
        int mState;
        ActivityInfo() {
            mActivity = null;
            mState = STATE_NONE;
        }
        ActivityInfo(Activity activity, int state) {
            mActivity = activity;
            mState = state;
        }
    }
    public void exitAllActivity() {
        if (null != mExistedActivitys) {
            // 先暂停监听（省得同时在2个地方操作列表）
            unregisterActivityLifecycleCallbacks( this );
            // 弹出的时候从头开始弹，和系统的 activity 堆栈保持一致
            for (ActivityInfo info : mExistedActivitys) {
                if (null == info || null == info.mActivity) {
                    continue;
                }
                try {
                    info.mActivity.finish();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            mExistedActivitys.clear();
            // 退出完之后再添加监听
            registerActivityLifecycleCallbacks( this );
        }
    }
    private ActivityInfo findActivityInfo(Activity activity) {
        if (null == activity || null == mExistedActivitys) {
            return null;
        }
        for (ActivityInfo info : mExistedActivitys) {
            if (null == info) {
                continue;
            }
            if (activity.equals(info.mActivity)) {
                return info;
            }
        }
        return null;
    }
}
    {% endhighlight %}

结束程序时调用：

    {% highlight java %}
BaseApplication.getInstance().exitAllActivity() ;
   {% endhighlight %}
   
但是如果应用内有多个进程，每创建一个进程就会跑一次Application的onCreate方法，每个进程内存都是独立的，
所以通过这种方式无法实现将应用的Activity放在同一个LinkedList中，不能实现完全退出一个应用。