---
layout: post
author: mxn
titile: wakelock解析
category: 技术博文
tag: android
---

在Android中,申请WakeLock可以让你的进程持续执行即使手机进入睡眠模式,比较实用的是比如:后台有网络功能,可以保证操作持续进行.

### 使用方法

在需要的地方开启WakeLock，需要结束的时候调用结束方法。

另外如果要使用WakeLock需要在Manifest中添加如下权限
<uses-permission android:name="android.permission.WAKE_LOCK" />

你可能还需要
<uses-permission android:name="android.permission.DEVICE_POWER"/>

WakeLock的设置是 Activiy 级别的，不是针对整个Application应用的。



    {% highlight java %}
private void acquireWakeLock() {
         if (wakeLock ==null) {
                PowerManager pm = (PowerManager) getSystemService(Context.POWER_SERVICE);
                wakeLock = pm.newWakeLock(PowerManager.PARTIAL_WAKE_LOCK, this.getClass().getCanonicalName());
                wakeLock.acquire();
            }
    }

private void releaseWakeLock() {
        if (wakeLock !=null&& wakeLock.isHeld()) {
            wakeLock.release();
            wakeLock =null;
        }
    }
    {% endhighlight %}

<!-- more -->


### PowerManager和WakeLock的操作步骤

1.PowerManager pm = (PowerManager) getSystemService(Context.POWER_SERVICE);通过 Context.getSystemService()
方法获取PowerManager实例。

2.然后通过PowerManager的newWakeLock((int flags, String tag)来生成WakeLock实例。int Flags指示要获取哪种WakeLock，
不同的Lock对cpu 、屏幕、键盘灯有不同影响。

3.获取WakeLock实例后通过acquire()获取相应的锁，然后进行其他业务逻辑的操作，最后使用release()释放（释放是必须的）。


### WakeLock的等级

newWakeLock((int flags, String tag)的第一个参数是WakeLock levelAndFlag，分别代表了一种WakeLock等级，并且可以通过「或」位操作组合使用。他们是：

1.PARTIAL_WAKE_LOCK：保证CPU保持高性能运行，而屏幕和键盘背光（也可能是触摸按键的背光）关闭。一般情况下都会使用这个WakeLock。

2.ACQUIRE_CAUSES_WAKEUP：这个WakeLock除了会使CPU高性能运行外还会导致屏幕亮起，即使屏幕原先处于关闭的状态下。

3.ON_AFTER_RELEASE：如果释放WakeLock的时候屏幕处于亮着的状态，则在释放WakeLock之后让屏幕再保持亮一小会。如果释放WakeLock的时候屏幕本身就没亮，则不会有动作。

#### 被弃用的WakeLock：

1.SCREEN_DIM_WAKE_LOCK：保证屏幕亮起，但是亮度可能比较低。同时键盘背光也可以不亮。

2.SCREEN_BRIGHT_WAKE_LOCK ：保证屏幕全亮，同时键盘背光也亮。

3.FULL_WAKE_LOCK：表现和SCREEN_BRIGHT_WAKE_LOCK 类似，但是区别在于这个等级的WakeLock使用的是最高亮度.

这三个Level在API17被弃用。被弃用说明肯定有替代品吗，上面三个类型的作用无非就是保持屏幕长亮。
所以推荐是用WindowFlagWindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON。使用方法是：
在Activity中： getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
甚至可以在布局中添加这个属性：android:keepScreenOn="true"


#### 被隐藏的WakeLock：

android中的部分api并不对用户应用开放，他们通过在注释中加入{@hide}来注明。但是在SDK提供的源代码中是可以看到的。

WAIT_FOR_PROXIMITY_NEGATIVE：用于和接近传感器配合使用，来实现电话应用中打电话时可以使屏幕黑掉，手机离开时又能使屏幕亮起来的功能。
acqure的时候屏幕会暗下来，release之后屏幕会亮。其值是32（int）。虽然被hide起来，但是在API21开放了.经试验，
不过newWakeLock的时候flag直接用32代替是可以创建这个等级的WakeLock的，但是因为是非开放API，不能保证第三方OEM修改这个代码实现.
因此使用起来并不安全。


### WakeLock常用方法

1.void acquire()：获取WakeLock

2.void acquire(long timeOut):获得WakeLock timeOut时长，当超过timeOut之后系统自动释放WakeLock。

3.release()：释放WakeLock

4.boolean isHeld()：判断是否已经获取WakeLock

5.void setReferenceCounted(boolean value)：是否使用引用计数。类似于垃圾回收策略，只是把垃圾回收改成了WakeLock回收。
如果value是true的话将启用该特性，如果一个WakeLock acquire了多次也必须release多次才能释放掉。但是如果释放次数比acquire
多则会抛出RuntimeException("WakeLock under-locked " + mTag)异常。默认是开启了引用计数的。

