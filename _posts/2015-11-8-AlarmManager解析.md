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

<!-- more -->

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

在上述2个步骤中，我们已经定义了activity布局和broadcast receiver。让我们看看MainActivity.java文件。在MyActivity类，
我们定义了三个简单的方法start()，stop()和startat10()。start()方法设置闹钟，cancel()方法取消预定的任务，
startAt10()方法将在10:30启动闹钟，并且固定30分钟间隔再次启动。

    {% highlight java %}

import android.app.Activity;
import android.app.AlarmManager;
import android.app.PendingIntent;
import android.content.Context;
import android.content.Intent;
import android.os.Bundle;
import android.view.View;
import android.widget.Toast;
import java.util.Calendar;

public class MyActivity extends Activity {

    private PendingIntent pendingIntent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_my);

        /* Retrieve a PendingIntent that will perform a broadcast */
        Intent alarmIntent = new Intent(MyActivity.this, AlarmReceiver.class);
        pendingIntent = PendingIntent.getBroadcast(MyActivity.this, 0, alarmIntent, 0);

        findViewById(R.id.startAlarm).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                start();
            }
        });

        findViewById(R.id.stopAlarm).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                cancel();
            }
        });

        findViewById(R.id.stopAlarmAt10).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                startAt10();
            }
        });
    }

    public void start() {
        AlarmManager manager = (AlarmManager) getSystemService(Context.ALARM_SERVICE);
        int interval = 8000;

        manager.setInexactRepeating(AlarmManager.RTC_WAKEUP, System.currentTimeMillis(), interval, pendingIntent);
        Toast.makeText(this, "Alarm Set", Toast.LENGTH_SHORT).show();
    }

    public void cancel() {
        AlarmManager manager = (AlarmManager) getSystemService(Context.ALARM_SERVICE);
        manager.cancel(pendingIntent);
        Toast.makeText(this, "Alarm Canceled", Toast.LENGTH_SHORT).show();
    }

    public void startAt10() {
        AlarmManager manager = (AlarmManager) getSystemService(Context.ALARM_SERVICE);
        int interval = 1000 * 60 * 30;

        /* Set the alarm to start at 10:30 AM */
        Calendar calendar = Calendar.getInstance();
        calendar.setTimeInMillis(System.currentTimeMillis());
        calendar.set(Calendar.HOUR_OF_DAY, 10);
        calendar.set(Calendar.MINUTE, 30);

        /* Repeating on every 30 minutes interval */
        manager.setRepeating(AlarmManager.RTC_WAKEUP, calendar.getTimeInMillis(),
                interval, pendingIntent);
    }

}
    {% endhighlight %}


### 开机自动启动服务

如前所述，一旦闹钟服务开始执行，它会一直执行直到它被停止或设备重启。这意味着，如果你的设备被重新启动，那么你的闹钟就停止了。
为了避免这种现状，你必须在设备启动完成后尽快重新启动闹钟服务。下面的代码将设置开启闹钟，一旦设备重启：

    {% highlight java %}
import android.app.AlarmManager;
import android.app.PendingIntent;
import android.content.BroadcastReceiver;
import android.content.Context;
import android.content.Intent;
import android.widget.Toast;

/**
 * @author Nilanchala
 *         <p/>
 *         Broadcast reciever, starts when the device gets starts.
 *         Start your repeating alarm here.
 */
public class DeviceBootReceiver extends BroadcastReceiver {

    @Override
    public void onReceive(Context context, Intent intent) {
        if (intent.getAction().equals("android.intent.action.BOOT_COMPLETED")) {
            /* Setting the alarm here */
            Intent alarmIntent = new Intent(context, AlarmReceiver.class);
            PendingIntent pendingIntent = PendingIntent.getBroadcast(context, 0, alarmIntent, 0);

            AlarmManager manager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
            int interval = 8000;
            manager.setInexactRepeating(AlarmManager.RTC_WAKEUP, System.currentTimeMillis(), interval, pendingIntent);

            Toast.makeText(context, "Alarm Set", Toast.LENGTH_SHORT).show();
        }
    }
}
    {% endhighlight %}

android系统在Manifest.permission中有这样一条RECEIVE_BOOT_COMPLETED的定义，当你自己的程序加入这个权限后，
就可以在系统启动完毕后收到一条系统的广播，这个广播的标志为ACTION_BOOT_COMPLETED，因此我们只要定义一个BroadcastReceiver
用来接收这个广播，然后加入自定义的动作即可。

接收不到BOOT_COMPLETED广播可能的原因:

1.BOOT_COMPLETED对应的action和uses-permission没有一起添加

2.应用安装到了sd卡内，安装在sd卡内的应用是收不到BOOT_COMPLETED广播的

3.系统开启了Fast Boot模式，这种模式下系统启动并不会发送BOOT_COMPLETED广播

4.应用程序安装后重来没有启动过，这种情况下应用程序接收不到任何广播，包括BOOT_COMPLETED、ACTION_PACKAGE_ADDED、
CONNECTIVITY_ACTION等等。

### 声明Application Manifest

为了在设备重启时启动闹钟服务，必须在application manifest中注册前面定义的DeviceBootReciever。同时还需要加入权限：
android.permission.RECEIVE_BOOT_COMPLETED。

    {% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.xxx.alarmservice" >

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >

        <!-- Permission to start Alarm on device reboot -->
        <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>

        <activity
            android:name=".MyActivity"
            android:label="@string/app_name" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <receiver android:name=".AlarmReceiver">
	      <intent-filter>
	           <action android:name="android.intent.action.BOOT_COMPLETED"/>
	      </intent-filter>
        </receiver>

        <!-- Will not be called unless the application explicitly enables it -->
        <receiver android:name=".DeviceBootReceiver"
            android:enabled="false">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
            </intent-filter>
        </receiver>

    </application>
</manifest>
    {% endhighlight %}


### AlarmManager常用方法总结

1.set(int type，long startTime，PendingIntent pi)；

该方法用于设置一次性闹钟，第一个参数表示闹钟类型，第二个参数表示闹钟执行时间，第三个参数表示闹钟响应动作。

2.setRepeating(int type，long triggerAtTime，long intervalTime，PendingIntent pi)；

该方法用于设置重复闹钟，第一个参数表示闹钟类型，第二个参数表示闹钟首次执行时间，第三个参数表示闹钟两次执行的间隔时间，
第三个参数表示闹钟响应动作。

3.setInexactRepeating（int type，long startTime，long intervalTime，PendingIntent pi）；

该方法也用于设置重复闹钟，与第二个方法相似，不过其两个闹钟执行的间隔时间不是固定的而已。


* 三个方法各个参数详悉：

1.int type： 闹钟的类型，常用的有5个值：AlarmManager.ELAPSED_REALTIME、 AlarmManager.ELAPSED_REALTIME_WAKEUP、
AlarmManager.RTC、 AlarmManager.RTC_WAKEUP、AlarmManager.POWER_OFF_WAKEUP。

AlarmManager.ELAPSED_REALTIME表示闹钟在手机睡眠状态下不可用，该状态下闹钟使用相对时间（相对于系统启动开始），状态值为3；

AlarmManager.ELAPSED_REALTIME_WAKEUP表示闹钟在睡眠状态下会唤醒系统并执行提示功能，该状态下闹钟也使用相对时间，状态值为2；

AlarmManager.RTC表示闹钟在睡眠状态下不可用，该状态下闹钟使用绝对时间，即当前系统时间，状态值为1；

AlarmManager.RTC_WAKEUP表示闹钟在睡眠状态下会唤醒系统并执行提示功能，该状态下闹钟使用绝对时间，状态值为0；

AlarmManager.POWER_OFF_WAKEUP表示闹钟在手机关机状态下也能正常进行提示功能，所以是5个状态中用的最多的状态之一，
该状态下闹钟也是用绝对时间，状态值为4；不过本状态好像受SDK版本影响，某些版本并不支持；

2.long startTime： 闹钟的第一次执行时间，以毫秒为单位，可以自定义时间，不过一般使用当前时间。需要注意的是，
本属性与第一个属性（type）密切相关，如果第一个参数对 应的闹钟使用的是相对时间（ELAPSED_REALTIME和ELAPSED_REALTIME_WAKEUP），
那么本属性就得使用相对时间（相对于 系统启动时间来说），比如当前时间就表示为：SystemClock.elapsedRealtime()；
如果第一个参数对应的闹钟使用的是绝对时间 （RTC、RTC_WAKEUP、POWER_OFF_WAKEUP），那么本属性就得使用绝对时间，
比如当前时间就表示 为：System.currentTimeMillis()。

3.long intervalTime：对于后两个方法来说，存在本属性，表示两次闹钟执行的间隔时间，也是以毫秒为单位。

4.PendingIntent pi： 绑定了闹钟的执行动作，比如发送一个广播、给出提示等等。PendingIntent是Intent的封装类。
需要注意的是，如果是通过启动服务来实现闹钟提示的话，PendingIntent对象的获取就应该采用Pending.getService(Context c,int i,Intent intent,int j)
方法；如果是通过广播来实现闹钟提示的话，PendingIntent对象的获取就应该采用 PendingIntent.getBroadcast(Context c,int i,Intent intent,int j)方法；
如果是采用Activity的方式来实现闹钟提示的话，PendingIntent对象的获取就应该采用 PendingIntent.getActivity(Context c,int i,Intent intent,int j)方法。
如果这三种方法错用了的话，虽然不会报错，但是看不到闹钟提示效果。

