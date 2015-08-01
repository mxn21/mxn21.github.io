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

    