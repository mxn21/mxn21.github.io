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
