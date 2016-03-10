---
layout: post
author: mxn
titile: 理解ThreadLocal
category: 技术博文
tag: android
---

ThreadLocal使用场合主要解决多线程中数据数据因并发产生不一致问题。ThreadLocal为每个线程的中并发访问的数据提供一个副本，
通过访问副本来运行业务，这样的结果是耗费了内存，单大大减少了线程同步所带来性能消耗，也减少了线程并发控制的复杂度。

ThreadLocal不能使用原子类型，只能使用Object类型。ThreadLocal的使用比synchronized要简单得多。
ThreadLocal和Synchonized都用于解决多线程并发访问。但是ThreadLocal与synchronized有本质的区 别。synchronized是利用锁的机制，
使变量或代码块在某一时该只能被一个线程访问。而ThreadLocal为每一个线程都提供了变量的副本， 使得每个线程在某一时间访问到的并不是同一个对象，
这样就隔离了多个线程对数据的数据共享。而Synchronized却正好相反，它用于在多个线程间通 信时能够获得数据共享。
Synchronized用于线程间的数据共享，而ThreadLocal则用于线程间的数据隔离。
当然ThreadLocal并不能替代synchronized,它们处理不同的问题域。Synchronized用于实现同步机制，比ThreadLocal更加复杂。

### ThreadLocal的接口方法

ThreadLocal类接口需要注意4个方法：




