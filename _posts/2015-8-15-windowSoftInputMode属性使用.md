---
layout: post
author: mxn
titile: windowSoftInputMode属性使用
category: 技术博文
tag: android
---

android:windowSoftInputMode表示Activity的主窗口如何与包含屏幕上的软键盘窗口交互。这个属性的设置将会影响两件事情 :

1.软键盘的状态,是否它是隐藏或显示,Activity成为用户关注的焦点。

2.Activity的主窗口调整,是否减少Activity主窗口大小以便腾出空间放软键盘或是否当Activity窗口的部分被软键盘覆盖时它的内容的当前焦点是可见的。

它的设置必须是下面列表中的一个值，或一个 ”state…”值加一个 ”adjust…”值的组合。


|              值          |                   作用  |
|            --------      |                -----   |
|     stateUnspecified     |软键盘的状态 (是否它是隐藏或可见 )没有被指定。系统将选择一个合适的状态或依赖于主题的设置。这个是为了软件盘行为默认的设置.|
|     stateUnchanged       | 软键盘被保持无论它上次是什么状态，是否可见或隐藏，当主窗口出现在前面时 |
|     stateHidden          |当用户选择该 Activity时，软键盘被隐藏——也就是，当用户确定导航到该 Activity时，而不是返回到它由于离开另一个 Activity |
|stateAlwaysHidden | 软键盘总是被隐藏的，当该 Activity主窗口获取焦点时 |
| stateVisible  |软键盘是可见的，当那个是正常合适的时 (当用户导航到 Activity主窗口时 ) |
|stateAlwaysVisible |当用户选择这个 Activity时，软键盘是可见的——也就是，也就是，当用户确定导航到该 Activity时，而不是返回到它由于离开另一个Activity。|
| adjustUnspecified | 它不被指定是否该 Activity主窗口调整大小以便留出软键盘的空间，或是否窗口上的内容得到屏幕上当前的焦点是可见的。系统将自动选择这些模式中一种主要依赖于是否窗口的内容有任何布局 视图能够滚动他们的内容。如果有这样的一个视图，这个窗口将调整大小，这样的假设可以使滚动窗口的内容在一个较小的区域中可见的。这个是主窗口默认的行为 设置。|
|adjustResize | 该Activity主窗口总是被调整屏幕的大小以便留出软键盘的空间|
| adjustPan | 该Activity主窗口并不调整屏幕的大小以便留出软键盘的空间。相反，当前窗口的内容将自动移动以便当前焦点从不被键盘覆盖和用户能总是看到输入内容的部分。这个通常是不期望比调整大小，因为用户可能关闭软键盘以便获得与被覆盖内容的交互操作。|

<!-- more -->

那么，这些取值到底是怎么影响到软键盘与窗口之间的交互的呢？下面，我们就一个个的测试这9个取值，到底是如何影响软键盘的显示的。

