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
     
你可以决定当一个异常发生时该发生什么样的事情，比如，使用StrictMode的penaltyLog()方法你可以在应用发生异常时查看adb logcat的输出。
当你发现一个比较严重的异常时，Android提供了一系列的工具来解决它：线程、Handler、AsyncTask、IntentService等等。但并不是StrictMode
报的所有问题都需要修复，特别是很多必须要在窗口生命周期回调中访问磁盘的时候。使用严苛模式可以帮你解决很多问题，比如在UI线程中访问网络始终
是一个问题。