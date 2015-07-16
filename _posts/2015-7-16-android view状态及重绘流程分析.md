---
layout: post
author: mxn
titile: android view状态及重绘流程分析
category: 技术博文
tag: android
---

### 视图状态

视图状态的种类非常多，一共有十几种类型，不过多数情况下我们只会使用到其中的几种，因此这里我们也就只去分析最常用的几种视图状态。

1. enabled

表示当前视图是否可用。可以调用setEnable()方法来改变视图的可用状态，传入true表示可用，传入false表示不可用。它们之间最大的区别在于，不可用的视图是无法响应onTouch事件的。

2. focused

表示当前视图是否获得到焦点。通常情况下有两种方法可以让视图获得焦点，即通过键盘的上下左右键切换视图，以及调用requestFocus()方法。而现在的Android手机几乎都没有键盘了，因此基本上只可以使用requestFocus()这个办法来让视图获得焦点了。而requestFocus()方法也不能保证一定可以让视图获得焦点，它会有一个布尔值的返回值，如果返回true说明获得焦点成功，返回false说明获得焦点失败。一般只有视图在focusable和focusable in touch mode同时成立的情况下才能成功获取焦点，比如说EditText。

3. window_focused

表示当前视图是否处于正在交互的窗口中，这个值由系统自动决定，应用程序不能进行改变。

4. selected

表示当前视图是否处于选中状态。一个界面当中可以有多个视图处于选中状态，调用setSelected()方法能够改变视图的选中状态，传入true表示选中，传入false表示未选中。

5. pressed

表示当前视图是否处于按下状态。可以调用setPressed()方法来对这一状态进行改变，传入true表示按下，传入false表示未按下。通常情况下这个状态都是由系统自动赋值的，但开发者也可以自己调用这个方法来进行改变。