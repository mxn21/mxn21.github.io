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

1.stateUnspecified

中文意思是未指定状态，当我们没有设置android:windowSoftInputMode属性的时候，软件默认采用的就是这种交互方式，
系统会根据界面采取相应的软键盘的显示模式，比如，当界面上只有文本和按钮的时候，软键盘就不会自动弹出，因为没有输入的必要。
那么，当界面上出现了获取了焦点的输入框的时候，软键盘会不会自动的弹出呢？这个还真不一定！比如，在下面的这个界面布局中，
软键盘并不会自动弹出。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img57.png)

就是说，默认的，在这种界面情况下，系统并不确定用户是否需要软键盘，因此不会自动弹出。但是，为什么说不一定呢？这是因为，如果我们在这个布局的外面，包裹上一个
ScrollView，软键盘就会自动的弹出来了！如下，在这种布局文件下，软键盘会自动的弹出

    {% highlight xml  %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical" >

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="match_parent" >

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical" >

            <Button
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:onClick="toOther"
                android:text="跳转" />

            <EditText
                android:layout_width="match_parent"
                android:layout_height="wrap_content" />
        </LinearLayout>
    </ScrollView>
</LinearLayout>
    {% endhighlight %}

这确实是一个很奇怪的判断方式。因此，我们可以得出结论，当设置属性为stateUnspecified的时候，系统是默认不弹出软键盘的，
但是当有获得焦点的输入框的界面有滚动的需求的时候，会自动弹出软键盘。至于为什么非要强调要获取焦点的输入框，这是因为，
如果不是输入框获取焦点，软键盘也是不会自动弹出的，让界面不自动弹出软键盘的其中一个解决方案，就是在xml文件中，
设置一个非输入框控件获取焦点，从而阻止键盘弹出。

2.stateUnchanged

中文的意思就是状态不改变的意思，我们应该怎么理解这句话呢？其实很好理解，就是说，当前界面的软键盘状态，
取决于上一个界面的软键盘状态。举个例子，假如当前界面键盘是隐藏的，那么跳转之后的界面，软键盘也是隐藏的；
如果当前界面是显示的，那么跳转之后的界面，软键盘也是显示状态。

3.stateHidden

顾名思义，如果我们设置了这个属性，那么键盘状态一定是隐藏的，不管上个界面什么状态，
也不管当前界面有没有输入的需求，反正就是不显示。因此，我们可以设置这个属性，来控制软键盘不自动的弹出。

4.stateAlwaysHidden

当该Activity主窗口获取焦点时，软键盘也总是被隐藏的

5.stateVisible

设置为这个属性，可以将软键盘召唤出来，即使在界面上没有输入框的情况下也可以强制召唤出来。


