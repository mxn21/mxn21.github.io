---
layout: post
author: mxn
titile: Android堆栈分析
category: 技术博文
tag: android
---

在开发中，与界面跳转联系比较紧密的概念是Task（任务）和Back Stack（回退栈）。activity的启动模式会影响Task和Back Stack的状态，
进而影响用户体验。除了启动模式之外，Intent类中定义的一些标志（以FLAG_ACTIVITY_开头）也会影响Task和Back Stack的状态。
在这篇文章中主要对android的堆栈管理进行分析和验证，其中涉及到activity的一个重要属性taskAffinity和Intent中的标志之一FLAG_ACTIVITY_NEW_TASK。

### Task和Back Stack简介

task是一组Activities的集合，一组Activities被Stack（back stack）所管理。

首先：我们来启动三个Activity来模拟生成活动与任务堆栈，三个Activity分别是：AndroidStackTaskActivity1、
AndroidStackTaskActivity2、AndroidStackTaskActivity3，具体代码如果下：

    {% highlight java  %}
    public class AndroidStackTaskActivity1 extends Activity implements OnClickListener{
        private Button next = null ;
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.main);
            init();
        }
        private void init(){
            next = (Button)findViewById(R.id.button1);
            next.setOnClickListener(this);
        }
        @Override
        public void onClick(View v) {
            Intent i = new Intent(this,AndroidStackTaskActivity2.class);
            startActivity(i);
        }
    }

    public class AndroidStackTaskActivity2 extends Activity  implements OnClickListener {
        private Button next = null ;
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.main2);
            init();
        }
        private void init(){
            next = (Button)findViewById(R.id.button1);
            next.setOnClickListener(this);
        }
        @Override
        public void onClick(View v) {
            // TODO Auto-generated method stub
            Intent i = new Intent(this,AndroidStackTaskActivity3.class);
            startActivity(i);
        }
    }

    public class AndroidStackTaskActivity3 extends Activity  implements OnClickListener {
        private Button next = null ;
        @Override
        public void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.main2);
            init();
        }
        private void init(){
            next = (Button)findViewById(R.id.button1);
            next.setOnClickListener(this);
        }
        @Override
        public void onClick(View v) {
            // TODO Auto-generated method stub
            Intent i = new Intent(this,AndroidStackTaskActivity3.class);
            startActivity(i);
        }
    }
    {% endhighlight %}

此时，生成的活动堆栈如下图所示：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img54.jpg)

首先activity1被start，此时，如果应用没有创建task则创建，并把activity1压入栈顶，activity1触发onCreate->onStart->onResume。

接着activity1转向到activity2时，activity1先触发onPause,activity2触发onCreate->onStart->onResume，然后activity1触发onPause->onStop，activity2压入栈顶。

以此类推，activity2转向activity3也是一样的步骤。那么当前栈顶是activity3。

当我们按下手机上的返回键时，栈顶的activity3触发onPause，activity2需要从状态stop到pause，所以触发了onPause->onStart->onResume，
activity3触发onStop->onDestory，因为activity3从栈顶弹出，所以触发onDestory，此时，activity2在栈顶。