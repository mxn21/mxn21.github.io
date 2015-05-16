---
layout: post
author: mxn
titile: RecyclerView使用
category: 技术博文
tag: android
---

首先是布局文件中使用 RecyclerView

{% highlight xml %}

	<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	tools:context=".MyActivity">
	
	<android.support.v7.widget.RecyclerView
	android:id="@+id/recyclerView"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:scrollbars="vertical" />
	
	</RelativeLayout>
{% endhighlight %}
