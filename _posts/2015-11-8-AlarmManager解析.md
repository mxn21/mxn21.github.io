---
layout: post
author: mxn
titile: alarmManager解析
category: 技术博文
tag: android
---

### AlarmManager的特点

1.AlarmManager是一种系统级别的定时提示服务，这允许app在一个给定的时间段上执行某一段代码。这可以运行在app的生命周期之外。

2.一旦设置AlarmManager，它将一直调用，即使你的应用程序是不运行，甚至当你的设备是在睡眠。

3.定时任务可以唤醒你的设备，但这会消耗更多的电池。因此，你必须小心那些执行频繁的定时任务。定时任务应该尽可能地设置低的频率。

4.一旦设置了AlarmManager，这将执行直到明确的调用停止方法或直到设备重启。

5.例如，一个AlarmManager用于启动长时间运行的操作，如从服务器每天一次同步数据。

6.AlarmManager允许你在给定的时间处理Intent

7.AlarmManager可以与broadcast receiver一起使用，以启动一个service来执行网络操作。

8.AlarmManager 与Timer，TimerTask不同，后者都执行在应用程序生命周期。


### 设置定时任务

type:闹钟类型，有四个可选值

1.ELAPSED_REALTIME：以手机开机的时间为基准

2.ELAPSED_REALTIME_WAKEUP：以手机开机的时间为基准，并且可以在休眠时发出广播

3.RTC：以UTC标准时间为基准

4.RTC_WAKEUP：以UTC标准时间为基准，并且可以在休眠时发出广播


