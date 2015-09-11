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

