---
layout: post
author: mxn
titile: FragmentTransactions和ActivityStateLoss
category: 技术博文
tag: android
---

有的时候我们会遇到如下错误，这篇文章就会分析出现这种错误的原因和解决方法。

    {% highlight java  %}
java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState
    at android.support.v4.app.FragmentManagerImpl.checkStateLoss(FragmentManager.java:1341)
    at android.support.v4.app.FragmentManagerImpl.enqueueAction(FragmentManager.java:1352)
    at android.support.v4.app.BackStackRecord.commitInternal(BackStackRecord.java:595)
    at android.support.v4.app.BackStackRecord.commit(BackStackRecord.java:574)

    {% endhighlight %}


### exception出现原因

抛出这个异常，是因为这activity的状态保存之后调用了commit FragmentTransaction，导致了一个 Activity state loss异常。
在分析这个问题之前，我们先看看onSaveInstanceState()这个方法被调用时会发生什么。我们需要知道Android applications在Android runtime
中并不能完全控制自己的生命周期。系统可以在任何时间结束一个app进程来释放内存。后台的activities可能在没有警告的情况下被杀死。
由于这些不稳定的行为是始终对用户隐藏的，所以framework给了每一个Activity一个机会，在销毁之前通过调用onSaveInstanceState()保存自己的状态。
当这些保存的状态。在以后进行重新装载的时候，用户需要感觉后台切换到前台是无缝的，而不管Activity有没有销毁。

当framework调用onSaveInstanceState()，需要传递一个Bundle对象，来保存Activity的状态，Activity在里面记录了dialogs，fragments
和views的状态。当方法return的时候，系统包装了Bundle对象，并且把这个对象通过Binder接口传递到系统服务进程，这那里它可以安全的保存。
当系统未来打算重建Activity时，系统重新把这个Bundle传递回application，用来使用它恢复Activity的旧的状态。

那么为什么会报出这个异常呢，这个错误阻止了onSaveInstanceState()被调用时Bundle保存Activity的内容。这意味着当在onSaveInstanceState()
之后调用FragmentTransaction#commit()时，transaction不会被保存，因为在一开始它没有作为Activity的状态被记录。


