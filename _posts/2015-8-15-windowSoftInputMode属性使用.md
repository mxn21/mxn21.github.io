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


6.stateAlwaysVisible

这个属性也是可以将键盘召唤出来，但是与stateVisible属性有小小的不同之处。举个例子，当我们设置为stateVisible属性，
如果当前的界面键盘是显示的，当我们点击按钮跳转到下个界面的时候，软键盘会因为输入框失去焦点而隐藏起来，
当我们再次回到当前界面的时候，键盘这个时候是隐藏的。但是如果我们设置为stateAlwaysVisible，我们跳转到下个界面，
软键盘还是隐藏的，但是当我们再次回来的时候，软键盘是会显示出来的。所以，这个Always就解释了这个区别，
不管什么情况到达当前界面(正常跳转或者是上一个界面被用户返回)，软键盘都是显示状态。


7.adjustUnspecified

从这个属性开始，就不是设置软键盘的显示与隐藏模式了，而是设置软键盘与软件的显示内容之间的显示关系。
当你跟我们没有设置这个值的时候，这个选项也是默认的设置模式。在这中情况下，系统会根据界面选择不同的模式。
如果界面里面有可以滚动的控件，比如ScrowView，系统会减小可以滚动的界面的大小，从而保证即使软键盘显示出来了，
也能够看到所有的内容。如果布局里面没有滚动的控件，那么软键盘可能就会盖住一些内容，我们从下面的图中可以看出差别。

没有滚动控件，软键盘下面的布局都被遮挡住了，若想修改，只能隐藏软键盘，然后选择。而且，重点注意一下上面的布局，
当我们选择的输入框偏下的时候，上面的标题栏和布局被软键盘顶上去了。记住这个特征，因为后面有个属性和这个的效果不一样。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img58.png)

布局里面有滑动控件，系统会自动的缩小整个界面的大小，因此，我们可以软键盘上面的小区域中显示所有的输入框。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img59.png)

这就是两中显示模式之间的差别。


8.adjustResize

这个属性表示Activity的主窗口总是会被调整大小，从而保证软键盘显示空间。我们先看显示效果。
注意观察这个上面的标题栏和按钮，设置为adjustResize属性之后，对于没有滑动控件的布局，虽然还是不能选择所有的输入框，
但是，窗口的显示方式发生了变化，默认属性时，整个布局是被顶上去了，但是设置为adjustResize属性，布局的位置并没有发生什么变化，
这就是最大的区别。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img60.png)

而对于有滑动控件的布局来说，显示效果和默认是一样的。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img61.png)


9.adjustPan

如果设置为这个属性，那么Activity的屏幕大小并不会调整来保证软键盘的空间，而是采取了另外一种策略，系统会通过布局的移动，
来保证用户要进行输入的输入框肯定在用户的视野范围里面，从而让用户可以看到自己输入的内容。对于没有滚动控件的布局来说，
这个其实就是默认的设置，如果我们选择的位置偏下，上面的标题栏和部分控件会被顶上去。但是对于有滚动控件的布局来说，
则不太一样，我们看下面的效果图。首先，这是软键盘没有弹出的时候，有滚动控件的显示范围，最下面显示的是9.

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img62.png)

当我们点击5这个输入框，我们会发现下面的现象。最上面只能够显示到按钮，标题栏已经不能看到了。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img63.png)

而最下面也只能滑动到8，下面的内容也不能够滑动了。

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img64.png)

因此，我们就能够理解这个属性的作用了。通过以上的实验，我们可以得出结论，如果我们不设置"adjust..."的属性，
对于没有滚动控件的布局来说，采用的是adjustPan方式，而对于有滚动控件的布局，则是采用的adjustResize方式。

了解了上面的这些知识之后，我们就可以根据自己的需求设置不同的方式了。而且，关于如何使得界面加载的时候不显示软键盘，
我们也有了一个很清楚的认识。

```注意：当设置属性：android:windowSoftInputMode = "adjustResize",软键盘弹出时，要对主窗口布局重新进行布局，并调用onSizeChanged方法.
而当我们设置android: windowSoftInputMode = "adjustPan"时，主窗口就不会调用onSizeChanged方法```

我们通过一段代码来测试一下，当我们设置了该属性后，弹出输入法时，系统做了什么：
重写Layout布局：

    {% highlight java  %}
public class ResizeLayout extends LinearLayout{
    private static int count = 0;
    public ResizeLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        Log.e("onSizeChanged " + count++, "=>onResize called! w="+w + ",h="+h+",oldw="+oldw+",oldh="+oldh);
    }
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        super.onLayout(changed, l, t, r, b);
        Log.e("onLayout " + count++, "=>OnLayout called! l=" + l + ", t=" + t + ",r=" + r + ",b="+b);
    }
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        Log.e("onMeasure " + count++, "=>onMeasure called! widthMeasureSpec=" + widthMeasureSpec + ", heightMeasureSpec=" + heightMeasureSpec);
    }
    {% endhighlight %}

我们的布局设置为：

    {% highlight xml  %}
<com.winuxxan.inputMethodTest.ResizeLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/root_layout"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical"
    >
    <EditText
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
    />
    <LinearLayout
            android:id="@+id/bottom_layout"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent"
            android:orientation="vertical"
            android:gravity="bottom">s
    <TextView
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:text="@string/hello"
        android:background="#77777777"
      />
   </LinearLayout>
</com.winuxxan.inputMethodTest.ResizeLayout>
    {% endhighlight %}

AndroidManifest.xml的Activity设置属性：android:windowSoftInputMode = "adjustResize"

运行程序，点击文本框，查看调试信息：

E/onMeasure 6(7960): =>onMeasure called! widthMeasureSpec=1073742144, heightMeasureSpec = 1073742024

E/onMeasure 7(7960): =>onMeasure called! widthMeasureSpec=1073742144, heightMeasureSpec = 1073742025

E/onSizeChanged 8(7960): =>onSizeChanged called! w=320,h=201,oldw=320,oldh=377

E/onLayout 9(7960): =>OnLayout called! l=0, t=0,r=320,b=201

从调试结果我们可以看出，当我们点击文本框后，根布局调用了onMeasure，onSizeChanged和onLayout。

windowSoftInputMode的值如果设置为adjustPan，那么该Activity主窗口并不调整屏幕的大小以便留出软键盘的空间。
相反，当前窗口的内容将自动移动以便当前焦点从不被键盘覆盖和用户能总是看到输入内容的部分。这个通常是不期望比调整大小，
因为用户可能关闭软键盘以便获得与被覆盖内容的交互操作。上面的例子中，
我们将AndroidManifest.xml的属性进行更改：android: windowSoftInputMode = "adjustPan"

重新运行，并点击文本框，查看调试信息：

E/onMeasure 6(8378): =>onMeasure called! widthMeasureSpec=1073742144, heightMeasureSpec=1073742200

E/onMeasure 7(8378): =>onMeasure called! widthMeasureSpec=1073742144, heightMeasureSpec=1073742201

E/onLayout 8(8378): =>OnLayout called! l=0, t=0,r=320,b=377

我们看到：系统也重新进行了measrue和layout，但是我们发现，layout过程中onSizeChanged并没有调用，
这说明输入法弹出前后并没有改变原有布局的大小。


