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
