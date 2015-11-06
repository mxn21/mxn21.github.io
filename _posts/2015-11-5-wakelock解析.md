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

#### 总结

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img124.png)

除了这四个Level之外，PowerMager还提供了两个Flag，可以配合Level使用。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img125.png)

注意: 这两个Flag和PARTIAL_WAKE_LOCK组合是没有作用的。

### WakeLock常用方法

1.void acquire()：获取WakeLock

2.void acquire(long timeOut):获得WakeLock timeOut时长，当超过timeOut之后系统自动释放WakeLock。

3.release()：释放WakeLock

4.boolean isHeld()：判断是否已经获取WakeLock

5.void setReferenceCounted(boolean value)：是否使用引用计数。类似于垃圾回收策略，只是把垃圾回收改成了WakeLock回收。
如果value是true的话将启用该特性，如果一个WakeLock acquire了多次也必须release多次才能释放掉。但是如果释放次数比acquire
多则会抛出RuntimeException("WakeLock under-locked " + mTag)异常。默认是开启了引用计数的。


### PowerManager的几个实用方法

1.boolean isScreenOn() 判断屏幕是否亮着（不管是暗的dimed还是正常亮度），在API20被弃用，推荐boolean isInteractive ()

2.void goToSleep(long time)time是时间戳，一般是System.currentTimeMillis()
+timeDelay。强制系统立刻休眠，需要Manifest中添加权限"android.permission.DEVICE_POWER"。按下电源键锁屏时调用的就是这个方法。

3.void wakeUp(long time)与上面对应。参数含义，所需权限与上同。按下电源键解锁屏幕时调用的就是这个方法。

4.void reboot(String reason)重启手机，reason是要传给linux内核的参数，比如“recovery”重启进recovery模式，“fastboot
”重启进fastboot模式。需要权限"android.permission.REBOOT"。


### 源码分析

看一下WakeLock的源码实现:

    {% highlight java %}
 public WakeLock newWakeLock(int levelAndFlags, String tag) {
        validateWakeLockParameters(levelAndFlags, tag);
        return new WakeLock(levelAndFlags, tag, mContext.getOpPackageName());
    }

    public static void validateWakeLockParameters(int levelAndFlags, String tag) {
        switch (levelAndFlags & WAKE_LOCK_LEVEL_MASK) {
            case PARTIAL_WAKE_LOCK:
            case SCREEN_DIM_WAKE_LOCK:
            case SCREEN_BRIGHT_WAKE_LOCK:
            case FULL_WAKE_LOCK:
            case PROXIMITY_SCREEN_OFF_WAKE_LOCK:
                break;
            default:
                throw new IllegalArgumentException("Must specify a valid wake lock level.");
        }
        if (tag == null) {
            throw new IllegalArgumentException("The tag must not be null.");
        }
    }
    {% endhighlight %}

newWakeLock方法首先检测LevelAndFlags和Tag的合法性：tag不能为空，Level必须用PowerManager提供的几个Level。
接下来，就进入到WakeLock的构造函数了。WakeLock是PowerManager的内部类。

    {% highlight java %}
    public final class WakeLock {
        private final int mFlags;
        private final String mTag;
        private final String mPackageName;
        private final IBinder mToken;
        private int mCount;
        private boolean mRefCounted = true;
        private boolean mHeld;

        private final Runnable mReleaser = new Runnable() {
            public void run() {
                release();
            }
        };

        WakeLock(int flags, String tag, String packageName) {
            mFlags = flags;
            mTag = tag;
            mPackageName = packageName;
            mToken = new Binder();
        }

        /**
         * Acquires the wake lock.
         * <p>
         * Ensures that the device is on at the level requested when the wake
         * lock was created.
         * </p>
         */
        public void acquire() {
            synchronized (mToken) {
                acquireLocked();
            }
        }

        private void acquireLocked() {
            if (!mRefCounted || mCount++ == 0) {
                // Do this even if the wake lock is already thought to be held
                // (mHeld == true)
                // because non-reference counted wake locks are not always
                // properly released.
                // For example, the keyguard's wake lock might be forcibly
                // released by the
                // power manager without the keyguard knowing. A subsequent call
                // to acquire
                // should immediately acquire the wake lock once again despite
                // never having
                // been explicitly released by the keyguard.
                mHandler.removeCallbacks(mReleaser);
                try {
                    mService.acquireWakeLock(mToken, mFlags, mTag, mPackageName, mWorkSource);
                } catch (RemoteException e) {
                }
                mHeld = true;
            }
        }

        /**
         * Releases the wake lock.
         * <p>
         * This method releases your claim to the CPU or screen being on. The
         * screen may turn off shortly after you release the wake lock, or it
         * may not if there are other wake locks still held.
         * </p>
         */
        public void release() {
            release(0);
        }

        /**
         * Releases the wake lock with flags to modify the release behavior.
         * <p>
         * This method releases your claim to the CPU or screen being on. The
         * screen may turn off shortly after you release the wake lock, or it
         * may not if there are other wake locks still held.
         * </p>
         *
         * @param flags
         *            Combination of flag values to modify the release behavior.
         *            Currently only {@link #WAIT_FOR_PROXIMITY_NEGATIVE} is
         *            supported.
         *
         *            {@hide}
         */
        public void release(int flags) {
            synchronized (mToken) {
                if (!mRefCounted || --mCount == 0) {
                    mHandler.removeCallbacks(mReleaser);
                    if (mHeld) {
                        try {
                            mService.releaseWakeLock(mToken, flags);
                        } catch (RemoteException e) {
                        }
                        mHeld = false;
                    }
                }
                if (mCount < 0) {
                    throw new RuntimeException("WakeLock under-locked " + mTag);
                }
            }
        }

        /**
         * Returns true if the wake lock has been acquired but not yet released.
         *
         * @return True if the wake lock is held.
         */
        public boolean isHeld() {
            synchronized (mToken) {
                return mHeld;
            }
        }
    }
        {% endhighlight %}

以acquire方法为例，通过对源码的分析，我们发现获取WakeLock的实现是通过mService进行的：

    {% highlight java %}
mService.acquireWakeLock(mToken, mFlags, mTag, mPackageName, mWorkSource);
     {% endhighlight %}

而mService是在PowerManager类里实例化的：

    {% highlight java %}
final IPowerManager mService;
     {% endhighlight %}

mService实例化类为/frameworks/base/services/java/com/android/server/power/PowerManagerService.java，
而到这里类里，你最终发现acquireWakeLock是由JNI层的native方法实现的。

    {% highlight java %}
private static native void nativeAcquireSuspendBlocker(String name);
     {% endhighlight %}

而这个方法的实现是在/frameworks/base/services/jni/com_android_server_power_PowerManagerService.cpp代码中：

    {% highlight c %}
static void nativeAcquireSuspendBlocker(JNIEnv *env, jclass clazz, jstring nameStr) {
    ScopedUtfChars name(env, nameStr);
    acquire_wake_lock(PARTIAL_WAKE_LOCK, name.c_str());
}
     {% endhighlight %}

这个acquire_wake_lock是在/hardware/libhardware_legacy/power/power.c里定义的


    {% highlight c %}
acquire_wake_lock(int lock, const char* id)
{
    initialize_fds();

//    ALOGI("acquire_wake_lock lock=%d id='%s'\n", lock, id);

    if (g_error) return g_error;

    int fd;

    if (lock == PARTIAL_WAKE_LOCK) {
        fd = g_fds[ACQUIRE_PARTIAL_WAKE_LOCK];
    }
    else {
        return EINVAL;
    }

    return write(fd, id, strlen(id));
}
     {% endhighlight %}

可以看到，acquire wake lock真正的实现是在fd所指向的文件中写了一串字符即可。
fd所指向的文件定义如下：

    {% highlight c %}
const char * const NEW_PATHS[] = {
    "/sys/power/wake_lock",
    "/sys/power/wake_unlock",
};
     {% endhighlight %}


### 其他用法扩展(官方API [Keeping the Device Awake](http://developer.android.com/intl/zh-cn/training/scheduling/wakelock.html#cpu) )

#### 保持屏幕点亮

某些应用程序需要保持屏幕打开，如游戏或电影应用程序。最好的办法是在你的activity中使用flag_keep_screen_on（必须只有一个activity，
不能是service或其他组件）。例如：

    {% highlight java %}

public class MainActivity extends Activity {
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    getWindow().addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON);
  }

    {% endhighlight %}

这种方法的优点是，不像akeLock，它不需要特殊的权限，由系统正确地管理用户在应用程序之间切换，不需要担心你的应用程序需要释放未使用的资源。

另一种方式来实现这是在你的应用程序的XML布局文件，用android:keepScreenOn属性：

    {% highlight xml %}
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:keepScreenOn="true">
    ...
</RelativeLayout>

    {% endhighlight %}

使用Android的keepscreenon=“true”相当于使用flag_keep_screen_on。你可以使用两者中适合自己的方法。设置flag的方式好处是，
它给你一个机会在稍后可以清除flag，从而使屏幕关闭。

注意：你不需要清除FLAG_KEEP_SCREEN_ON，除非你不再想让屏幕点亮（例如，一段闲置时间后设置超时）。window manager负责确保当应用程序进入后台或返回到前台时，
能够正确运行。但如果你想清除flag，从而让屏幕关闭，使用clearflags():

    {% highlight java %}
getWindow().clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON).
    {% endhighlight %}


#### 保持CPU运行

如果你需要保存CPU运转，在设备进入睡眠之前完成一些工作，你可以使用PowerManager系统服务功能：WakeLock。WakeLock允许您的应用程序来控制设备的电源状态。
创建和保持WakeLock会对设备的电池时间产生巨大的影响。因此，你应该只有在必要的时候使用WakeLock，并使用尽可能短的时间。
例如，你不应该在Activity的整个生命周期中使用WakeLock。上面提到过，如果你想在activity中保持屏幕点亮，用flag_keep_screen_on。

使用WakeLock的一个合法的案例可能是一个后台service，需要用WakeLock使屏幕关闭时保持CPU进行工作。虽然，这种做法应尽量减少，
因为其对电池寿命的影响。

使用WakeLock，第一步是在manifest中添加wake_lock权限：

    {% highlight xml %}
<uses-permission android:name="android.permission.WAKE_LOCK" />
    {% endhighlight %}

如果你的应用程序包括一个broadcast receiver，使用一个service做了一些工作，你可以通过WakefulBroadcastReceiver管理你的wake lock
(可以参考下面对WakefulBroadcastReceiver的讲解),这是首选方法。如果你的app没有遵循这个模式，这里是如何设置一个wake lock ：


