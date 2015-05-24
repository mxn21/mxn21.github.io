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

Activity 中
{% highlight java %}
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_my);

        RecyclerView recyclerView = (RecyclerView) findViewById(R.id.recyclerView);

        // 创建一个线性布局管理器
        LinearLayoutManager layoutManager = new LinearLayoutManager(this);
        // 设置横向显示
        //layoutManager.setOrientation(LinearLayoutManager.HORIZONTAL);  
        // 设置布局管理器
        recyclerView.setLayoutManager(layoutManager);

        // 创建数据集
        String[] dataset = new String[100];
        for (int i = 0; i < dataset.length; i++){
            dataset[i] = "item" + i;
        }
        // 创建Adapter，并指定数据集
        MyAdapter adapter = new MyAdapter(dataset);
        // 设置Adapter
        recyclerView.setAdapter(adapter);

    }

{% endhighlight %}
