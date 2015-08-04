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

task是可以跨应用的，这正是task存在的一个重要原因。有的Activity，虽然不在同一个app中，但为了保持用户操作的连贯性，
把他们放在同一个任务中。例如，在我们的应用中的一个Activity A中点击发送邮件，会启动邮件程序的一个Activity B来发送邮件，
这两个activity是存在于不同app中的，但是被系统放在一个任务中，这样当发送完邮件后，用户按back键返回，可以返回到原来的Activity A中，
这样就确保了用户体验。

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

设置了这个flag，如果要启动的Activity在当前任务中已经存在了，就不会再次创建这个Activity的实例，
而是会把这个Activity之上的所有Activity全部关闭掉。比如说，一个任务当中有A、B、C、D四个Activity，
然后D调用了startActivity()方法来启动B，并将flag指定成FLAG_ACTIVITY_CLEAR_TOP，那么此时C和D就会被关闭掉，
现在返回栈中就只剩下A和B了。
那么此时Activity B会接收到这个启动它的Intent，你可以决定是让Activity B调用onNewIntent()方法(不会创建新的实例)，
还是将Activity B销毁掉并重新创建实例。如果Activity B没有在manifest中指定任何启动模式(也就是"standard"模式)，
并且Intent中也没有加入一个FLAG_ACTIVITY_SINGLE_TOP flag，那么此时Activity B就会销毁掉，然后重新创建实例。
而如果Activity B在manifest中指定了任何一种启动模式，或者是在Intent中加入了一个FLAG_ACTIVITY_SINGLE_TOP flag，
那么就会调用Activity B的onNewIntent()方法。

```FLAG_ACTIVITY_CLEAR_TOP和FLAG_ACTIVITY_NEW_TASK结合在一起使用也会有比较好的效果，比如可以将一个后台运行的任务切换到前台，
并把目标Activity之上的其它Activity全部关闭掉。这个功能在某些情况下非常有用，比如说从通知栏启动Activity的时候。```

以上只介绍了常有用的几种控制Activity跳转的Flag标识.

### 主要的<activity>属性

主要的<activity>属性有：
launchMode.
taskAffinity.
allowTaskReparenting.
alwaysRetainTaskState.
clearTaskOnLaunch .
finishOnTaskLaunch.

下面将对每一个属性和标志一一介绍：

#### affinity

通常来说一个程序内/任务栈中的Activity具有亲和力，也就是说具有相同亲和力的Activity默认属于同一个任务Task中.

affinity可以用于指定一个Activity更加愿意依附于哪一个任务，在默认情况下，同一个应用程序中的所有Activity都具有相同的affinity，
所以，这些Activity都更加倾向于运行在相同的任务当中。当然了，你也可以去改变每个Activity的affinity值，
通过<activity>元素的taskAffinity属性就可以实现了。

taskAffinity属性接收一个字符串参数，你可以指定成任意的值(字符串中至少要包含一个.)，
但必须不能和应用程序的包名相同，因为系统会使用包名来作为默认的affinity值。

```
affinity决定两件事情——Activity重新宿主（从一个Task跳到了另一个Task中，新的Task就被称为重新宿主）的Task（参考allowTaskReparenting特性）
和使用FLAG_ACTIVITY_NEW_TASK标志启动的Activity宿主的Task。
  注意：affinity只有在加载activity的Intent对象包含了FLAG_ACTIVITY_NEW_TASK 标记，或者当activity的allowTaskReparenting属性设置为“true”时才有效。
```

affinity主要有以下应用场景：
当调用startActivity()方法来启动一个Activity时，默认是将它放入到当前的任务当中。但是，
如果在Intent中加入了一个FLAG_ACTIVITY_NEW_TASK flag的话(或者该Activity在manifest文件中声明的启动模式是"singleTask")，
系统就会尝试为这个Activity单独创建一个任务。但是规则并不是只有这么简单，系统会去检测要启动的这个Activity的affinity和当前任务的affinity是否相同，
如果相同的话就会把它放入到现有任务当中，如果不同则会去创建一个新的任务。而同一个程序中所有Activity的affinity默认都是相同的，
这也是前面为什么说，同一个应用程序中即使声明成"singleTask"，也不会为这个Activity再去创建一个新的任务了。

#### allowTaskReparenting

当把Activity的allowTaskReparenting属性设置成true时，Activity就拥有了一个转移所在任务的能力。
具体点来说，就是一个Activity现在是处于某个任务当中的，但是它与另外一个任务具有相同的affinity值，
那么当另外这个任务切换到前台的时候，该Activity就可以转移到现在的这个任务当中。

那还是举一个形象点的例子吧，比如有一个天气预报程序，它有一个Activity是专门用于显示天气信息的，
这个Activity和该天气预报程序的所有其它Activity具体相同的affinity值，并且还将allowTaskReparenting属性设置成true了。
这个时候，你自己的应用程序通过Intent去启动了这个用于显示天气信息的Activity，
那么此时这个Activity应该是和你的应用程序是在同一个任务当中的。但是当把天气预报程序切换到前台的时候，
这个Activity又会被转移到天气预报程序的任务当中，并显示出来，因为它们拥有相同的affinity值，
并且将allowTaskReparenting属性设置成了true。

如何用户将任务切换到后台之后过了很长一段时间，系统会将这个任务中除了最底层的那个Activity之外的其它所有Activity全部清除掉。
当用户重新回到这个任务的时候，最底层的那个Activity将得到恢复。这个是系统默认的行为，因为既然过了这么长的一段时间，
用户很有可能早就忘记了当时正在做什么，那么重新回到这个任务的时候，基本上应该是要去做点新的事情了。

当然，既然说是默认的行为，那就说明我们肯定是有办法来改变的，在<activity>元素中设置以下几种属性就可以改变系统这一默认行为：

#### alwaysRetainTaskState

如果将最底层的那个Activity的这个属性设置为true，那么上面所描述的默认行为就将不会发生，任务中所有的Activity
即使过了很长一段时间之后仍然会被继续保留。

一般来说，特定的情形如当用户从主画面重新选择这个Task时，系统会对这个Task进行清理（从stack中删除位于根Activity之上的所有Activivity）。
典型的情况，当用户有一段时间没有访问这个Task时也会这么做，例如30分钟。
然而，当这个特性设为“true”时，用户总是能回到这个Task的最新状态，无论他们是如何启动的。这非常有用，
例如，像Browser应用程序，这里有很多的状态（例如多个打开的Tab），用户不想丢失这些状态。系统会为我们保持这些状态数据。


#### clearTaskOnLaunch

如果将最底层的那个Activity的这个属性设置为true，那么只要用户离开了当前任务，
再次返回的时候就会将最底层Activity之上的所有其它Activity全部清除掉。简单来讲，就是一种和alwaysRetainTaskState完全相反的工作模式，
它保证每次返回任务的时候都会是一种初始化状态，即使用户仅仅离开了很短的一段时间。
这个特性只对启动一个新的Task的Activity（根Activity）有意义；对Task中其它的Activity忽略。

假设，某人从主画面启动了ActivityP，并从那里迁移至Activity Q。接下来用户按下HOME，然后返回Activity P。
一般，用户可能见到的是Activity Q，因为它是P的Task中最后工作的内容。然而，如果P设定这个特性为“true”，
当用户按下HOME并使这个Task再次进入前台时，其上的所有的Activity(在这里是Q)都将被清除。因此，当返回到这个Task时，
用户只能看到P。如果这个特性和allowTaskReparenting都设定为“true”，那些能重新宿主的Activity会移动到共享affinity的Task中；
剩下的Activity都将被抛弃。

#### finishOnTaskLaunch

这个属性和clearTaskOnLaunch是比较类似的，不过它不是作用于整个任务上的，而是作用于单个Activity上。
如果某个Activity将这个属性设置成true，那么用户一旦离开了当前任务，再次返回时这个Activity就会被清除掉。
如果这个特性和allowTaskReparenting都设定为“true”，这个特性胜出，Activity的affinity忽略。这个Activity不会重新宿主，但是会销毁。


