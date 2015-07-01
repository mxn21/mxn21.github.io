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