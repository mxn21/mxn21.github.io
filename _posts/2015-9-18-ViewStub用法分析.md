---
layout: post
author: mxn
titile: ViewStub用法分析
category: 技术博文
tag: android
---

ViewStub组件和<include>标签的作用类似，主要是为了提高布局的重用性，及布局的模块化。它们之间最大的差别是，
ViewStub中的布局不会随着它所在布局的渲染而渲染，而<include>标签中的布局会随着它所在布局的渲染而渲染，
ViewStub中的布局只有在你需要的时候才会渲染到主界面中。

ViewStub是一个看不见的，轻量级的View。它没有尺寸，也不会绘制以及以某种形式参与到布局中来。这意味着ViewStub去inflate以及
保留在View层次中的代价是很廉价的。ViewStub最佳的描述称之为“懒惰的include”。ViewStub中引用的布局只在你想添加到UI上时才会显示。

### 使用方法

1.在ButtonOne与ButtonTwo之间存在一个ViewStub布局，如下图：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img103.jpg)

2.单击ButtonOne后渲染ViewStub中的布局，如下图：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img104.jpg)

3.实现代码：

为了使用ViewStub，你所有需要做的是指定android:id特性，便于以后inflate，指定android:layout特性，
引用布局文件。ViewStub还允许你使用第三个特性，android:inflatedId，你可以使用它来重写包含的布局文件中的根元素的id。
最后，在ViewStub上设定的layout_*参数将会应用到包含的布局文件的顶部。


res/layout/main.xml实现：

    {% highlight xml  %}

<?xml version="1.0" encoding="utf-8"?>

<LinearLayout xmlns:android = "http://schemas.android.com/apk/res/android"
    android:orientation = "vertical"
    android:layout_width = "fill_parent"
    android:layout_height = "fill_parent"
    >

    <Button
        android:id = "@+id/show"
        android:text = "ButtonOne"
        android:layout_width = "wrap_content"
        android:layout_height = "wrap_content"
        />

    <ViewStub
        android:id = "@+id/viewStub"
        android:layout = "@layout/green_layout"
        android:layout_width = "300dip"
        android:layout_height = "300dip"
        />

    <Button
        android:layout_width = "wrap_content"
        android:layout_height = "wrap_content"
        android:text = "ButtonTwo"
        />

</LinearLayout>

    {% endhighlight %}

main.xml中ViewStub组件里的布局实现：

    {% highlight xml  %}

<?xml version="1.0" encoding="utf-8"?>

<LinearLayout
    xmlns:android = "http://schemas.android.com/apk/res/android"
    android:layout_width = "match_parent"
    android:layout_height = "match_parent"
    android:background = "@color/green">

</LinearLayout>

    {% endhighlight %}


主Activity实现：

    {% highlight java  %}

public class ViewStubActivity extends Activity {

    private ViewStub mViewStub;

    private Button mShow;

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        mViewStub = (ViewStub) findViewById(R.id.viewStub);

        mShow = (Button) findViewById(R.id.show);
        mShow.setOnClickListener(new OnClickListener() {
            public void onClick(View view) {
                if (mViewStub != null) {
                    mViewStub.inflate();
                }
            }
        });
    }
}

    {% endhighlight %}

你准备inflate ViewStub时，调用inflate()方法即可。你还可以设定ViewStub的Visibility为VISIBLE或INVISIBLE，
也会触发inflate。注意的是，使用inflate()方法能返回布局文件的根View。

有一点需要记住的是：当ViewStub inflate后，这个ViewStub就从View层次中移除了。因此，没有必要保留一个对ViewStub的引用。
ViewStub是快捷编程与高效编程之间的产物。与其手动的inflate View并在运行时添加到View层次上，不如简单的使用ViewStub。
它相当“廉价”且易于使用。ViewStub唯一的缺点是现在不支持<merge />标签。

在使用ViewStub的过程中，有一点需要特别注意。对于一个ViewStub而言，当setVisibility(int)或inflate()方法被调用之后，
这个ViewStub在布局中将被使用指定的View替换，所以inflate过一遍的ViewStub，如果被隐藏之后再次想要显示，
将不能使用inflate()方法，但是可以再次使用setVisibility(int)方法设置为可见，这就是这两个方法的区别。
而inflate()被调用之后，返回的是父布局控件对象,ViewStub对象会被置为空。按句话说，某个被ViewStub指定的布局被Inflate后，就不会够再通过ViewStub来控制它了。
因此，如果多次对ViewStub进行infalte，会出现错误信息：ViewStub must have a non-null ViewGroup viewParent。

### 总结

1.当布局文件inflate时，ViewStub控件虽然也占据内存，但是相相比于其他控件，ViewStub所占内存很小；

2.布局文件inflate时，ViewStub主要是作为一个“占位符”的性质，放置于view tree中，且ViewStub本身是不可见的。
ViewStub中有一个layout属性，指向ViewStub本身可能被替换掉的布局文件，在一定时机时，通过viewStub.inflate()完成此过程；

3.ViewStub本身是不可见的，对ViewStub setVisibility(..)与其他控件不一样，ViewStub的setVisibility 成View.VISIBLE或INVISIBLE
如果是首次使用，都会自动inflate其指向的布局文件，并替换ViewStub本身，再次使用则是相当于对其指向的布局文件设置可见性。

4.前面讲到的ViewStub指向的布局文件解析inflate并替换掉当前ViewStub本身，并不是完全意义上的替换（与include标签还不太一样），
替换时，布局文件的layout params是以ViewStub为准，其他布局属性是以布局文件自身为准。

