---
layout: post
author: mxn
titile: wakelock解析
category: 技术博文
tag: android
---

在Android中,申请WakeLock可以让你的进程持续执行即使手机进入睡眠模式,比较实用的是比如:后台有网络功能,可以保证操作持续进行.

使用方法：在需要的地方开启WakeLock，需要结束的时候调用结束方法。

    {% highlight java %}
private void acquireWakeLock() {
         if (wakeLock ==null) {
                PowerManager pm = (PowerManager) getSystemService(Context.POWER_SERVICE);
                wakeLock = pm.newWakeLock(PowerManager.SCREEN_DIM_WAKE_LOCK, this.getClass().getCanonicalName());
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


待续.
