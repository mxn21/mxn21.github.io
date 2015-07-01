---
layout: post
author: mxn
titile: 重新认识activity生命周期
category: 技术博文
tag: android
---

重新认识activity生命周期

#### onWindowFocusChanged重要作用

Activity生命周期中，onStart, onResume, onCreate都不是真正visible的时间点，
真正的visible时间点是onWindowFocusChanged()函数被执行时。从onWindowFocusChanged被执行起，用户可以与应用进行交互了，
而这之前，对用户的操作需要做一点限制。

这个onWindowFocusChanged指的是这个Activity得到或者失去焦点的时候 就会被调用
也就是说 如果你想要做一个Activity一加载完毕，就触发什么的话 完全可以用这个。

使用一个view的getWidth() getHeight() 方法来获取该view的宽和高，返回的值却为0。
如果这个view的长宽很确定不为0的话，那很可能是你过早的调用这些方法，也就是说在这个view被加入到rootview之前你就调用了这些方法，返回的值自然为0.
解决该问题的方法有很多，主要就是延后调用这些方法。可以试着在onWindowFocusChanged()里面调用这些方法,验证时可以获取到View的宽高的。

看官方API:android API :

void onWindowFocusChanged(boolean hasFocus)

Called when the current Window of the activity gains or loses focus.

可以看出在窗口获得焦点或失去时调用



```注意：onWindowFocusChanged当Activity被创建时是在onResume之后被调用，当Activity被覆盖或者退居后台或者当前Activity退出时，它是在onPause之后被调用```

<!-- more -->


下面的demo解释了整个生命周期：


	{% highlight java  %}

	public class LifeCycleActivity extends Activity {

        private static final String TAG = "LifeCycleActivity";
        private Context context = this;
        private int param = 1;

        //Activity创建时被调用
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            Log.i(TAG, "onCreate called.");

            setContentView(R.layout.lifecycle);

            Button btn = (Button) findViewById(R.id.btn);
            btn.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    Intent intent = new Intent(context, TargetActivity.class);
                    startActivity(intent);
                }
            });
        }

        //Activity创建或者从后台重新回到前台时被调用
        @Override
        protected void onStart() {
            super.onStart();
            Log.i(TAG, "onStart called.");
        }

        //Activity从后台重新回到前台时被调用
        @Override
        protected void onRestart() {
            super.onRestart();
            Log.i(TAG, "onRestart called.");
        }

        //Activity创建或者从被覆盖、后台重新回到前台时被调用
        @Override
        protected void onResume() {
            super.onResume();
            Log.i(TAG, "onResume called.");
        }

        //Activity窗口获得或失去焦点时被调用,在onResume之后或onPause之后
        @Override
        public void onWindowFocusChanged(boolean hasFocus) {
            super.onWindowFocusChanged(hasFocus);
            Log.i(TAG, "onWindowFocusChanged called.");
        }

        //Activity被覆盖到下面或者锁屏时被调用
        @Override
        protected void onPause() {
            super.onPause();
            Log.i(TAG, "onPause called.");
            //有可能在执行完onPause或onStop后,系统资源紧张将Activity杀死,所以有必要在此保存持久数据
        }

        //退出当前Activity或者跳转到新Activity时被调用
        @Override
        protected void onStop() {
            super.onStop();
            Log.i(TAG, "onStop called.");
        }

        //退出当前Activity时被调用,调用之后Activity就结束了
        @Override
        protected void onDestroy() {
            super.onDestroy();
            Log.i(TAG, "onDestory called.");
        }

        /**
         * Activity被系统杀死时被调用.
         * 例如:屏幕方向改变时,Activity被销毁再重建;当前Activity处于后台,系统资源紧张将其杀死.
         * 另外,当跳转到其他Activity或者按Home键回到主屏时该方法也会被调用,系统是为了保存当前View组件的状态.
         * 在onPause之前被调用.
         */
        @Override
        protected void onSaveInstanceState(Bundle outState) {
            outState.putInt("param", param);
            Log.i(TAG, "onSaveInstanceState called. put param: " + param);
            super.onSaveInstanceState(outState);
        }

        /**
         * Activity被系统杀死后再重建时被调用.
         * 例如:屏幕方向改变时,Activity被销毁再重建;当前Activity处于后台,系统资源紧张将其杀死,用户又启动该Activity.
         * 这两种情况下onRestoreInstanceState都会被调用,在onStart之后.
         */
        @Override
        protected void onRestoreInstanceState(Bundle savedInstanceState) {
            param = savedInstanceState.getInt("param");
            Log.i(TAG, "onRestoreInstanceState called. get param: " + param);
            super.onRestoreInstanceState(savedInstanceState);
        }
    }

	{% endhighlight %}

#### onSaveInstanceState和onRestoreInstanceState

注意到，除了几个常见的方法外，我们还添加了onWindowFocusChanged、onSaveInstanceState、onRestoreInstanceState方法：

1.onWindowFocusChanged方法：在Activity窗口获得或失去焦点时被调用，例如创建时首次呈现在用户面前；当前Activity被其他Activity覆盖；
当前Activity转到其他Activity或按Home键回到主屏，自身退居后台；用户退出当前Activity。以上几种情况都会调用onWindowFocusChanged，
并且当Activity被创建时是在onResume之后被调用，当Activity被覆盖或者退居后台或者当前Activity退出时，它是在onPause之后被调用，如图所示：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img12.png)

这个方法在某种场合下还是很有用的，例如程序启动时想要获取视特定视图组件的尺寸大小，在onCreate中可能无法取到，因为窗口Window
对象还没创建完成，这个时候我们就需要在onWindowFocusChanged里获取.

2.onSaveInstanceState：
(1)在Activity被覆盖或退居后台之后，系统资源不足,将其杀死，此方法会被调用；
(2)在用户改变屏幕方向时，此方法会被调用；
(3)在当前Activity跳转到其他Activity或者按Home键回到主屏，自身退居后台时，此方法会被调用。第一种情况我们无法保证什么时候发生，系统根据资源紧张程度去调度；第二种是屏幕翻转方向时，系统先销毁当前的Activity，
然后再重建一个新的，调用此方法时，我们可以保存一些临时数据；第三种情况系统调用此方法是为了保存当前窗口各个View组件的状态。onSaveInstanceState的调用顺序是在onPause之前。

3.onRestoreInstanceState：
(1)在Activity被覆盖或退居后台之后，系统资源不足将其杀死，然后用户又回到了此Activity，此方法会被调用；
(2)在用户改变屏幕方向时，重建的过程中，此方法会被调用。我们可以重写此方法，以便可以恢复一些临时数据。onRestoreInstanceState的调用顺序是在onStart之后。


下面我们就来操作一下这个Activity，看看它的生命周期到底是个什么样的过程：

1.启动Activity：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img13.png)

在系统调用了onCreate和onStart之后，调用了onResume，自此，Activity进入了运行状态。

2.跳转到其他Activity，或按下Home键回到主屏：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img14.png)

我们看到，此时onSaveInstanceState方法在onPause之前被调用了，并且注意，退居后台时，onPause后onStop相继被调用。

3.从后台回到前台：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img15.png)


当从后台会到前台时，系统先调用onRestart方法，然后调用onStart方法，最后调用onResume方法，Activity又进入了运行状态。

4.修改TargetActivity在AndroidManifest.xml中的配置，将android:theme属性设置为@android:style/Theme.Dialog，
然后再点击LifeCycleActivity中的按钮，跳转行为就变为了TargetActivity覆盖到LifeCycleActivity之上了，此时调用的方法为：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img16.png)

注意还有一种情况就是，我们点击按钮，只是按下锁屏键，执行的效果也是如上。

我们注意到，此时LifeCycleActivity的OnPause方法被调用，并没有调用onStop方法，因为此时的LifeCycleActivity没有退居后台，
只是被覆盖或被锁屏；onSaveInstanceState会在onPause之前被调用。

```注意：按锁屏键和按HOME键的生命周期是不同的，按HOME执行onPause --> onStop --> onRestart --> onStart ---> onResume```
```按锁屏键和对话框覆盖界面的生命周期是一样的，都只会进行onPause －－> onResume .不会执行onStop,onRestart,onStart ```


5.按回退键使LifeCycleActivity从被覆盖回到前面，或者按解锁键解锁屏幕：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img17.png)


此时只有onResume方法被调用，直接再次进入运行状态。

6.退出：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img18.png)


最后onDestory方法被调用，标志着LifeCycleActivity的终结。

大家似乎注意到，在所有的过程中，并没有onRestoreInstanceState的出现，这个并不奇怪，因为之前我们就说过，
onRestoreInstanceState只有在杀死不在前台的Activity之后用户回到此Activity，或者用户改变屏幕方向的这两个重建过程中被调用。我们要演示第一种情况比较困难，
我们可以结合第二种情况演示一下具体过程。顺便也向大家讲解一下屏幕方向改变的应对策略。

首先介绍一下关于Activity屏幕方向的相关知识。
我们可以为一个Activity指定一个特定的方向，指定之后即使转动屏幕方向，显示方向也不会跟着改变：

1.指定为竖屏：在AndroidManifest.xml中对指定的Activity设置android:screenOrientation="portrait"，或者在onCreate方法中指定：

setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);  //竖屏

2.指定为横屏：在AndroidManifest.xml中对指定的Activity设置android:screenOrientation="landscape"，或者在onCreate方法中指定：

setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_LANDSCAPE); //横屏

这是应用中的Activity设置特定的方向是经常用到的办法，可以为我们省去不少不必要的麻烦。不过，我们今天讲的是屏幕方向改变时的生命周期，所以我们并不采用固定屏幕方向这种办法。
下面我们就结合实例讲解一下屏幕转换的生命周期，我们新建一个Activity命名为OrientationActivity，如下：


	{% highlight java  %}

	public class OrientationActivity extends Activity {

        private static final String TAG = "OrientationActivity";
        private int param = 1;

        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.orientation_portrait);
            Log.i(TAG, "onCreate called.");
        }

        @Override
        protected void onStart() {
            super.onStart();
            Log.i(TAG, "onStart called.");
        }

        @Override
        protected void onRestart() {
            super.onRestart();
            Log.i(TAG, "onRestart called.");
        }

        @Override
        protected void onResume() {
            super.onResume();
            Log.i(TAG, "onResume called.");
        }

        @Override
        protected void onPause() {
            super.onPause();
            Log.i(TAG, "onPause called.");
        }

        @Override
        protected void onStop() {
            super.onStop();
            Log.i(TAG, "onStop called.");
        }

        @Override
        protected void onDestroy() {
            super.onDestroy();
            Log.i(TAG, "onDestory called.");
        }

        @Override
        protected void onSaveInstanceState(Bundle outState) {
            outState.putInt("param", param);
            Log.i(TAG, "onSaveInstanceState called. put param: " + param);
            super.onSaveInstanceState(outState);
        }

        @Override
        protected void onRestoreInstanceState(Bundle savedInstanceState) {
            param = savedInstanceState.getInt("param");
            Log.i(TAG, "onRestoreInstanceState called. get param: " + param);
            super.onRestoreInstanceState(savedInstanceState);
        }

        //当指定了android:configChanges="orientation"后,方向改变时onConfigurationChanged被调用
        @Override
        public void onConfigurationChanged(Configuration newConfig) {
            super.onConfigurationChanged(newConfig);
            Log.i(TAG, "onConfigurationChanged called.");
            switch (newConfig.orientation) {
            case Configuration.ORIENTATION_PORTRAIT:
                setContentView(R.layout.orientation_portrait);
                break;
            case Configuration.ORIENTATION_LANDSCAPE:
                setContentView(R.layout.orientation_landscape);
                break;
            }
        }
    }

	{% endhighlight %}


当我们旋转屏幕时，我们发现系统会先将当前Activity销毁，然后重建一个新的：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img19.png)


系统先是调用onSaveInstanceState方法，我们保存了一个临时参数到Bundle对象里面，然后当Activity重建之后我们又成功的取出了这个参数。
为了避免这样销毁重建的过程，我们需要在AndroidMainfest.xml中对OrientationActivity对应的<activity>配置android:configChanges="orientation"，然后我们再测试一下，
我试着做了四次的旋转，打印如下：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img20.png)

可以看到，每次旋转方向时，只有onConfigurationChanged方法被调用，没有了销毁重建的过程。

以下是需要注意的几点：

1.如果<activity>配置了android:screenOrientation属性，则会使android:configChanges="orientation"失效。

2.模拟器与真机差别很大：模拟器中如果不配置android:configChanges属性或配置值为orientation，切到横屏执行一次销毁->重建，切到竖屏执行两次。
真机均为一次。模拟其中如果配置android:configChanges="orientation|keyboardHidden"，切竖屏执行一次onConfigurationChanged，切竖屏执行两次。真机均为一次。


#### 重新认识生命周期

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img21.png)


```上表中 “是否可以终止”是指系统是否可以在此方法执行完后直接销毁该Activity的实例（没有再去执行此Activity内的其他代码）。
如上表所示，onPause(),onStop(),和onDestroy()三个方法是可以终止的。同时从状态图（上图）中可以知道onPause()方法是这三个方法中第一个调用的，
自一个Activity创建开始，直到此Activity终止或销毁，onPause()是最后一个保证会被调用的方法（如在突发情况下系统回收内存，onStop()和onDestroy()就不会被调用）。
因此，你应该在onPause()方法里保存一些关键的数据（如用户编辑的内容），所谓“关键“就是让你有选择地去保存一些重要的数据，因为onPause()方法执行时间过长，会延迟下一状态的切换，进而影响了用户的体验。
在上表注明不可以被终止的方法也可能会被系统终止，不过这一情况只会在一些极端的情况下发生。
同时可以联想到service的生命周期中没有onPause，所以想在service生命周期中判断service被中止，是比较棘手的```


#### 保存Activity的状态信息(onSaveInstanceState和OnRestoreInstanceState的具体使用)

在暂停状态下的activity，因其activity信息完整保存在内存里且保持与窗口管理器的连接，所以可以直接调用onRestart（）方法还原activity。
用户经常会使用返回按钮返回上一个界面（Activity），用户当然是希望上一个界面（Activity）和他原来看到或编辑的一样，在这种情况下你就需要使用之前已经保存好的状态信息和成员变量来还原上一个Activity。
若要保存一个Activity的状态信息和成员变量，则需要实现回调方法onSaveInstanceState()。onSaveInstanceState()会在执行onPause()
方法之前调用，这个方法有一个参数Bundle，可以以“名称-值”的形式保存activity的信息
（如使用putString(),putInt()方法）。接着在需要还原activity时，系统会调用onCreat()或者OnRestoreInstanceState()，这两个方法都传入一个以保存了activity信息的Bundle对象，通过提取Bundle对象的信息来恢复activity。
如果没有信息需要保存到Bundle对象，那传递给这两个方法的将是空的Bundle对象（刚开始初始化一个activity时其实就是这种情况）

其实，即使你没有实现回调方法onSaveInstanceState()，系统也会执行默认的onSaveInstanceState()。值得注意的是，
系统会为每个View组件执行相对应的onSaveInstanceState()，这可以使每一个组件保存自己相对应的信息。在Android里，
几乎所有的控件都实现了这个方法，因此在界面产生的可见变化都会自动保存下来，也就可以还原activity。举个例子：
在EditText输入的内容或者CheckBox的选择，都会自动保存下来。而你所需要做的就是为需要保存信息的控件提供一个id（在
XML文件里的属性android:id）,如果你没有为控件提供id，那系统是不会自动保存这些信息的。
下面给出一些例子参考一下onSaveInstanceState()和OnRestoreInstanceState()方法的使用：


	{% highlight java  %}

    @Override
    protected void OnSaveInstanceState(Bundle outState)
    {
        outState.PutString("myString", "HelloXamarin.Android OnSaveInstanceState");
        outState.PutBoolean("myBool", true);
        base.OnSaveInstanceState(outState);
    }
    @Override
    protected void OnRestoreInstanceState(Bundle savedState)
    {
        base.OnRestoreSaveInstanceState(savedState);
        var myString =savedState.GetString(amyStringa);
        var myBool =GetBoolean(amyBoola);
    }

    {% endhighlight %}


如果需要强制控件不保存对应的信息，可以在布局文件设置android:saveEnabled = “false”，或者使用控件对象的setSaveEnabled()方法。

尽管系统默认执行onSaveInstanceState()保存对应控件的信息，有时候你需要实现这个方法去保存一些额外的变量。例如：你需要去保存一些在这个activity生命周期里会改变的变量（指控件信息之外的变量），这些会改变的变量系统是默认不保存的。

需要注意的是：onSaveInstanceState()方法不保证一定会被调用，所以在这个方法里你只应该保存一些临时变量（如UI的状态），而不要去保存一些需要持久化的数据。相反，你应该在onPaused（）方法里保存需要持久化的数据（例如保存到数据库里的数据）。


```Android系统上Back按钮和Home按钮的区别，尽管他们都可以导航到另外一个otherActivity（可以是其他程序的），
可他们的作用原理是不一样的。在界面（Activity）上点击Back按钮，系统会认为这个Activity已完成其对应的实现逻辑，
则调用对应的方法销毁此Activity；可当点击Home按钮时，系统会认为这个Activity还没有完成对应的逻辑，则系统并不会销毁这个Activity，
而是让其处于停止态（Stopped State）。```

