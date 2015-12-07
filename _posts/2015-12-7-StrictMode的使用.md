---
layout: post
author: mxn
titile: StrictMode的使用
category: 技术博文
tag: android
---

StrictMode通常用于抓取在应用程序的主线程中来操作磁盘或者网络访问的问题，比如界面操作和动画，在非UI线程中处理磁盘文件和网络
操作使得程序更快、响应更及时，在保证你的程序足够流程的同时，你还需要阻止ANR事件的发生。

下面是启用StrictMode的实例：

      {% highlight java  %}
 private void setStrictMode() {
        if (Integer.valueOf(Build.VERSION.SDK) > 3) {
            Log.d(LOG_TAG, "Enabling StrictMode policy over Sample application");
            StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
                    .detectAll()
                    .penaltyLog()
                    .penaltyDeath()
                    .build());
            StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
                    .detectAll()
                    .penaltyLog()
                    .build());
        }
    }
     {% endhighlight %}
     
 