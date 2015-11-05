---
layout: post
author: mxn
titile: wakelock解析
category: 技术博文
tag: android
---

在Android中,申请WakeLock可以让你的进程持续执行即使手机进入睡眠模式,比较实用的是比如:后台有网络功能,可以保证操作持续进行.

使用方法：在需要的地方开启WakeLock，需要结束的时候调用结束方法。

另外如果要使用WakeLock需要在Manifest中添加如下权限
<uses-permission android:name="android.permission.WAKE_LOCK" />


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


PowerManager和WakeLock的操作步骤
　　PowerManager pm = (PowerManager) getSystemService(Context.POWER_SERVICE);通过 Context.getSystemService().方法获取PowerManager实例。
　　然后通过PowerManager的newWakeLock((int flags, String tag)来生成WakeLock实例。int Flags指示要获取哪种WakeLock，不同的Lock对cpu 、屏幕、键盘灯有不同影响。
　　获取WakeLock实例后通过acquire()获取相应的锁，然后进行其他业务逻辑的操作，最后使用release()释放（释放是必须的）。







待续.
