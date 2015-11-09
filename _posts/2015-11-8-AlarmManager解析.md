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

4.RTC_WAKEUP：以UTC标准时间为基准，并且可以在休眠时发出广播。这种方式是最常见的形式。

下面写一个例子，使用RTC类型。该应用程序使用一个activity，包含三个按钮。一个按钮是为启动定时服务，另一个是取消预定服务。
第三个按钮是在指定的日历时间开始闹钟。例如，如果你设置为你的生日提醒。它将调用在10:30，并将在每30分钟后重复。

下面看一下布局文件：

    {% highlight xml %}

<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context=".MyActivity">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dp"
        android:text="@string/hint"
        android:textAppearance="?android:attr/textAppearanceLarge" />

    <Button
        android:id="@+id/startAlarm"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentLeft="true"
        android:layout_alignParentStart="true"
        android:layout_below="@+id/textView"
        android:layout_marginBottom="20dp"
        android:text="Start Alarm Service" />

    <Button
        android:id="@+id/stopAlarm"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/startAlarm"
        android:layout_marginBottom="20dp"
        android:text="Stop Alarm" />


    <Button
        android:id="@+id/stopAlarmAt10"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@+id/stopAlarm"
        android:text="Stop Alarm at 10:30" />

</RelativeLayout>

    {% endhighlight %}

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img127.png)


### 定义定时BroadcastReceiver

在这个例子中，我们将与broadcast receiver关联。定时服务将在预定时间调用这个接收器。为了简单起见，每一次的任务我们只是向用户
展示了一个toast。您可以编写您的逻辑来启动服务或下载任务。

如果你的闹钟有执行网络任务，可用在broadcast的onRecieve()方法中start下载service。

    {% highlight java %}
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

public class AlarmReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {

        // For our recurring task, we'll just display a message
        Toast.makeText(context, "I'm running", Toast.LENGTH_SHORT).show();
    }
}

    {% endhighlight %}

### 在activity中开始和停止任务
