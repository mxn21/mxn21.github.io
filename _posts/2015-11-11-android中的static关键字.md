---
layout: post
author: mxn
titile: android中的static关键字
category: 技术博文
tag: android
---


### android中static变量的生命周期

Android的静态变量的生命周期遵守Java的设计。我们知道静态变量是在类被load的时候分配内存的，并且存在于方法区。当类被卸载的时候，
静态变量被销毁。在java中，一个类被加载和卸载，可简单的等同于jvm进程的启动和结束。在Android中就是启动app的时候被加载，中止app的
时候被卸载，注意必须是被完全关闭，而不是进入后台，也不是pause。

几点注意：

三、Android中的进程什么时候结束

这个是Android对进程和内存管理不同于PC的核心——如果资源足够，Android不会杀掉任何进程，
另一个意思就是进程随时可能会被杀掉。而Android会在资源够的时候，重启被杀掉的进程。也就是说静态变量的值，
如果不做处理，是不可靠的，可以说内存中的一切都不可靠。如果要可靠，还是得保存到Nand或SD卡中去，在重启的时候恢复回来。

另一种情况就是不能把退出所有Activity等同于进程的退出，所以在用户点击图标启动应用的时候，以前存放于静态变量中的值，
有可能还存在，因此要视具体情况给予清空操作。

四、Application也是一样不可靠

Application其实是一个单例对象，也是放在内存中的，当进程被杀掉，就全清空了，只不过Android系统会帮重建Application，
而我们存放在Application的数据自然就没有了，还是得自己处理。

五、静态引用的对象不会被垃圾回收

只要静态变量没有被销毁也没有置null，其对象一直被保持引用，也即引用计数不可能是0，因此不会被垃圾回收。因此，单例对象在运行时不会被回收。





### 在android中不要用static声明view

在android中不要用staitc声明view（包括view的子类）,因为在创建视图的时候会使用LayoutInflater.inflate()来导入视图，
setContentView(R.layout.ID)底层同样是使用inflate导入视图，这需要使用一个当前的Context，并且传递给被导入的视图。
那么view视图就会一直持有这个context的引用，这个context可以通过view.getContext()来取出。

因此，view持有一个activty实例，用static声明view会让view无法被回收，那么view持有的activiy实例也就不能回收，从而导致内存泄漏.


