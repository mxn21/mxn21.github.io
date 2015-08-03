---
layout: post
author: mxn
titile: Android堆栈分析
category: 技术博文
tag: android
---

在开发中，与界面跳转联系比较紧密的概念是Task（任务）和Back Stack（回退栈）。activity的启动模式会影响Task和Back Stack的状态，
进而影响用户体验。除了启动模式之外，Intent类中定义的一些标志（以FLAG_ACTIVITY_开头）也会影响Task和Back Stack的状态。
在这篇文章中主要对android的堆栈管理进行分析和验证，其中涉及到activity的一个重要属性taskAffinity和Intent中的标志之一FLAG_ACTIVITY_NEW_TASK。

### Task和Back Stack简介

task(任务)是一组Activities的集合，一组Activities被Stack（back stack）所管理,栈中Activity的顺序就是按照它们被打开的顺序依次存放的。

手机的Home界面是大多数task开始的地方，当用户在Home界面上点击了一个应用的图标时，这个应用的task就会被转移到前台。
如果这个应用目前并没有任何一个任务的话(说明这个应用最近没有被启动过)，系统就会去创建一个新的task，
并且将该应用的主Activity放入到返回栈当中。

当一个Activity启动了另外一个Activity的时候，新的Activity就会被放置到返回栈的栈顶并将获得焦点。
前一个Activity仍然保留在返回栈当中，但会处于停止状态。当用户按下Back键的时候，栈中最顶端的Activity会被移除掉，
然后前一个Activity则会得重新回到最顶端的位置。返回栈中的Activity的顺序永远都不会发生改变，
我们只能向栈顶添加Activity，或者将栈顶的Activity移除掉。因此，返回栈是一个典型的后进先出(last in, first out)的数据结构。

<!-- more -->

下面来用代码做一个验证：
首先：我们来启动三个Activity来模拟生成活动与任务堆栈，三个Activity分别是：AndroidStackTaskActivity1、
AndroidStackTaskActivity2、AndroidStackTaskActivity3，具体代码如果下：

    {% highlight java  %}
    public class AndroidStackTaskActivity1 extends Activity implements OnClickListener{
        private Button next = null ;
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.main);
            init();
        }
        private void init(){
            next = (Button)findViewById(R.id.button1);
            next.setOnClickListener(this);
        }
        @Override
        public void onClick(View v) {
            Intent i = new Intent(this,AndroidStackTaskActivity2.class);
            startActivity(i);
        }
    }

    public class AndroidStackTaskActivity2 extends Activity  implements OnClickListener {
        private Button next = null ;
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.main2);
            init();
        }
        private void init(){
            next = (Button)findViewById(R.id.button1);
            next.setOnClickListener(this);
        }
        @Override
        public void onClick(View v) {
            // TODO Auto-generated method stub
            Intent i = new Intent(this,AndroidStackTaskActivity3.class);
            startActivity(i);
        }
    }

    public class AndroidStackTaskActivity3 extends Activity  implements OnClickListener {
        private Button next = null ;
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.main2);
            init();
        }
        private void init(){
            next = (Button)findViewById(R.id.button1);
            next.setOnClickListener(this);
        }
        @Override
        public void onClick(View v) {
            // TODO Auto-generated method stub
            Intent i = new Intent(this,AndroidStackTaskActivity3.class);
            startActivity(i);
        }
    }
    {% endhighlight %}


此时，生成的活动堆栈如下图所示：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img54.jpg)

首先activity1被start，此时，如果应用没有创建task则创建，并把activity1压入栈顶，activity1触发onCreate->onStart->onResume。

接着activity1转向到activity2时，activity1先触发onPause,activity2触发onCreate->onStart->onResume，然后activity1触发onPause->onStop，activity2压入栈顶。

以此类推，activity2转向activity3也是一样的步骤。那么当前栈顶是activity3。

当我们按下手机上的返回键时，栈顶的activity3触发onPause，activity2需要从状态stop到pause，所以触发了onPause->onStart->onResume，
activity3触发onStop->onDestory，因为activity3从栈顶弹出，所以触发onDestory，此时，activity2在栈顶。

如果继续按返回键，当前栈顶的activity弹出并被destory，直到home界面。当所有的activity都弹出了，这个task也就消亡了。


当开始一个新的task时，前一个task被设置为后台。在后台，所有的activity都处理stop状态，但是back stack保留了所有后台activity的状态信息，只是丢失了焦点。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img55.png)

这个时候，用户还可以将任意后台的任务切换到前台，这样用户应该就会看到之前离开这个task时处于最顶端的那个Activity。

由于返回栈中的Activity的顺序永远都不会发生改变，所以如果你的应用程序中允许有多个入口都可以启动同一个Activity，
那么每次启动的时候就都会创建该Activity的一个新的实例，而不是将下面的Activity的移动到栈顶。这样的话就容易导致一个问题的产生，
即同一个Activity有可能会被实例化很多次，如下图所示：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img56.png)


但是呢，如果你不希望同一个Activity可以被多次实例化，这些功能甚至更多功能，
都是可以通过在manifest文件中设置<activity>元素的属性，或者是在启动Activity时配置Intent的flag来实现的。

下面我们就将开始讨论，如何通过manifest参数，以及Intent flag来改变Activity在任务中的默认行为。

1.使用manifest文件

当你在manifest文件中声明一个Activity的时候，你可以指定这个Activity在启动的时候该如何与任务进行关联。

2.使用Intent flag

当你调用startActivity()方法时，你可以在Intent中加入一个flag，从而指定新启动的Activity该如何与当前任务进行关联。
也就是说，如果Activity A启动了Activity B，Activity B可以定义自己该如何与当前任务进行关联，
而Activity A也可以要求Activity B该如何与当前任务进行关联。如果Activity B在manifest中已经定义了该如何与任务进行关联，
而Activity A同时也在Intent中要求了Activity B该怎么样与当前任务进行关联，那么此时Intent中的定义将覆盖manifest中的定义。

```需要注意的是，有些启动模式在manifest中可以指定，但在Intent中是指定不了的。
同样，也有些启动模式在Intent中可以指定，但在manifest中是指定不了的.```

### 使用manifest文件

当在manifest文件中定义Activity的时候，你可以通过<activity>元素的launchMode属性来指定这个Activity应该如何与任务进行关联。
launchMode属性一共有以下四种可选参数：standard,singleTop,singleTask,singleInstance.
之前已经详细介绍过这些内容，这里不在分析。

### 使用Intent flags

除了使用manifest文件之外，你也可以在调用startActivity()方法的时候，为Intent加入一个flag来改变Activity与任务的关联方式，
下面我们来一一讲解一下每种flag的作用：

#### 1.FLAG_ACTIVITY_NEW_TASK

设置了这个flag，新启动Activity就会被放置到一个新的task当中(与"singleTask"有点类似，但不完全一样)
，当然这里讨论的仍然还是启动其它程序中的Activity。这个flag的作用通常是模拟一种Launcher的行为，
即列出一推可以启动的东西，但启动的每一个Activity都是在运行在自己独立的任务当中的。

#### 2.FLAG_ACTIVITY_SINGLE_TOP

设置了这个flag，如果要启动的Activity在当前任务中已经存在了，并且还处于栈顶的位置，那么就不会再次创建这个Activity的实例，
而是直接调用它的onNewIntent()方法。这种flag和在launchMode中指定"singleTop"模式所实现的效果是一样的。

#### 3.FLAG_ACTIVITY_CLEAR_TOP








