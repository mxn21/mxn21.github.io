---
layout: post
author: mxn
titile: Async Task解析
category: 技术博文
tag: android
---

Android UI是线程不安全的，如果想要在子线程里进行UI操作，就需要借助Android的异步消息处理机制。不过为了更加方便我们在子线程中更新UI元素，Android从1.5版本就引入了一个AsyncTask类，使用它就可以非常灵活方便地从子线程切换到UI线程。

## 分析AsyncTask的源码

在启动某一个任务之前，要先new出它的实例，因此，先来看一看AsyncTask构造函数中的源码，如下所示：
