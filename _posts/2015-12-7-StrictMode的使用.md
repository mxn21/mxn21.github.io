---
layout: post
author: mxn
titile: StrictMode的使用
category: 技术博文
tag: android
---

StrictMode通常用于抓取在应用程序的主线程中来操作磁盘或者网络访问的问题，比如界面操作和动画，在非UI线程中处理磁盘文件和网络
操作使得程序更快、响应更及时，在保证你的程序足够流程的同时，你还需要阻止ANR事件的发生。 ANR窗口产生的原因是多种多样的。
程序的主线程因为IO读写或网络阻塞而导致被阻塞了，外部存储设备被独占了或系统负荷过高都可能导致ANR。
从Android 2.3开始提供了一个新的类StrictMode，可以帮助开发者改进他们的Android应用，StrictMode可以用于捕捉发生在应用程序
主线程中耗时的磁盘、网络访问或函数调用，使主线程处理UI和动画在磁盘读写和网络操作时变得更平滑，避免主线程被阻塞，导致ANR窗口的发生。

下面是启用StrictMode的实例，可以在Application的OnCreate中添加，这样就能在程序启动的最初一刻进行监控了。

      {% highlight java  %}
 private void setStrictMode() {
        if (Integer.valueOf(Build.VERSION.SDK) > 3) {
            Log.d(LOG_TAG, "Enabling StrictMode policy over Sample application");
            StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
                    .detectAll()    // 这里可以替换为.detectDiskReads().detectDiskWrites().detectNetwork()。
                                    // detectAll() 包括了磁盘读写和网络I/O
                    .penaltyLog()   //打印logcat，当然也可以定位到dropbox，通过文件保存相应的log
                    .penaltyDeath()
                    .build());
            StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
                    .detectAll()
                    .penaltyLog()
                    .build());
        }
    }
     {% endhighlight %}
     
StrictMode 共有两种策略(policy)：

1.ThreadPolicy: 线程相关策略，包括主线程访问网络、磁盘(现在手机中使用闪存)读写、慢代码的检测。我们能够分别检测(detect)
这些操作或允许(permit)这些操作。一旦出现了违规(violation)，就会有相应的提示（比如 Log 显示）。我们可以使用 detectNetwork()
检测主线程网络访问，detectDiskReads() 和 detectDiskWrites() 检测主线程磁盘读写，detectCustomSlowCalls()
检测主线程自定义的慢代码。当然也可以使用 permitXXX() 允许这些操作。

2.VMPolicy: 虚拟机相关的策略，包括 SQLite 或 SQLiteCursor 没关闭、实现 Closable 接口的类使用后没关闭 等。
我们可以使用detectLeakedSqliteObjects() 检测 SQLite 和 SQLiteCursor 内存泄漏(没关闭)，detectLeakedClosableObjects()
检测实现 Closable 接口的对象内存泄漏。

我们能通过 StrictMode.getThreadPolicy() 和 StrictMode.getVMPolicy() 获得当前采取的 ThreadPolicy 和 VMPolicy。

你可以决定当一个异常发生时该发生什么样的事情，比如，使用StrictMode的penaltyLog()方法你可以在应用发生异常时查看adb logcat的输出。
penaltyLog()表示将警告输出到LogCat，你也可以使用其他或增加新的惩罚（penalty）函数，例如使用penaltyDeath()的话，一旦StrictMode消息被
写到LogCat后应用就会崩溃。和ThreadPolicy不同的是，VmPolicy不能通过一个对话框提供警告。

因为设置发生在线程中，严苛模式（StrictMode）甚至能在从一个对象到另一个对象的控制流中找到违例事件。当违例发生，
你会惊奇地注意到代码正运行于主线程，而栈trace将帮助你发现它如何发生。于是你能单步调试解决问题，或是将代码移到它自己的后台线程，
或是就保持原来的处理方式。这都取决与你。当然，你可能希望适时关闭严苛模式（StrictMode），当你的程序作为产品发布时，
你可不希望它仅为了一个警告在你的用户手里崩溃。有两个方法可以关闭严苛模式（StrictMode），最直接的就是移除相应代码，
但这样做不利于持续开发的产品。你通常可以定义一个应用级别布尔变量来测试是否需要调用严苛模式（StrictMode）代码。
在发布产品前将这个值定义为FALSE。更优雅的方式是利用调试模式（debug mode）的特点，在AndroidManifest.xml中定义这个布尔变量。
<application>字段的属性之一是android:debuggable，其义自明。
    
    
当你发现一个比较严重的异常时，Android提供了一系列的工具来解决它：线程、Handler、AsyncTask、IntentService等等。但并不是StrictMode
报的所有问题都需要修复，特别是很多必须要在窗口生命周期回调中访问磁盘的时候。使用严苛模式可以帮你解决很多问题，比如在UI线程中访问网络始终
是一个问题。