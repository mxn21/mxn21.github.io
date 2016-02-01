---
layout: post
author: mxn
titile: TextView超链接实现方式总结
category: 技术博文
tag: android
---

TextView中的超链接可以通过几种方式实现：1.Html.fromHtml，2.Spannable，3.Linkify.addLinks。下面分别进行测试，包括
修改字体样式，下划线样式，点击事件等。


### Html.fromHtml方式

TextView，本身就支持部分的Html格式标签。这其中包括常用的字体大小颜色设置，文本链接等。使用起来也比较方便，只需要使用Html类转换一下即可：
textView.setText(Html.fromHtml(str)); 

代码如下：

      {% highlight java  %} 
public class Test10Activity extends Activity {

    TextView textView ;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test10);
        textView = (TextView) findViewById(R.id.text);
        String webLinkText = "<a href='https://souly.cn'> html超链接测试</a>" ;
        textView.setText(Html.fromHtml(webLinkText));
    }
}

      {% endhighlight %}
      
效果如下：

//======================

这时候点击超链接并没有跳转效果，需要加上 

      {% highlight java  %} 
textView.setMovementMethod(LinkMovementMethod.getInstance());
      {% endhighlight %}
      
之后才会跳转。
这样点击之后会用默认浏览器打开url。

现在字体颜色和下划线是默认样式，修改字体颜色比较简单，可以直接用<font>标签：

      {% highlight java  %} 
 String webLinkText =
          "<font color='#333333'><a href='https://souly.cn' style='text-decoration:none; color:#0000FF'> 
          html超链接测试</a>" ;
      {% endhighlight %}

但是想不借助其他类去掉下划线就没有办法了。

还有一种更简单的方法就是在TextView的xml布局中加入autoLink自动识别，这样做操作最简单，但是也不能修改样式：

      {% highlight xml  %} 
      
     <TextView
           android:id="@+id/text"
           android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:textSize="20sp"
           android:text="souly.cn"
           android:autoLink="email|phone|web"
           />
           
      {% endhighlight %}

### Spannable方式

在xml中设置了android:autoLink="email|phone|web"后，url文字下面会有一条下划线,我们可以看URLSpan所继承的类ClickableSpan
类的源码，如下：

      {% highlight java %} 
      public abstract class ClickableSpan extends CharacterStyle implements UpdateAppearance {  
        
          /** 
           * Performs the click action associated with this span. 
           */  
          public abstract void onClick(View widget);  
           
          /** 
           * Makes the text underlined and in the link color. 
           */  
          @Override  
          public void updateDrawState(TextPaint ds) {  
              ds.setColor(ds.linkColor);  
              ds.setUnderlineText(true);  
          }  
      }  
       {% endhighlight %}
       
它调用ds.setUnderlineText(true); 设置了下划线。我们再设置一个没有下划线的Spannable对象就可以了。
我们重写一个类继承UnderlineSpan，和ClickableSpan一样，都是CharacterStyle的子类。

使用方法如下：

      {% highlight java %} 
public class Test10Activity extends Activity {
    
    TextView textView ;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test10);
        textView = (TextView) findViewById(R.id.text);

        String webLinkText = "我的博客-->https://souly.cn" ;
        textView.setText(webLinkText);
        NoUnderlineSpan mNoUnderlineSpan = new NoUnderlineSpan();
        if (textView.getText() instanceof Spannable) {
            Spannable s = (Spannable) textView.getText();
            s.setSpan(mNoUnderlineSpan, 0, s.length(), Spanned.SPAN_MARK_MARK);
        }
    }

    public static class NoUnderlineSpan extends UnderlineSpan {
        public NoUnderlineSpan() {}
        public NoUnderlineSpan(Parcel src) {}
        @Override
        public void updateDrawState(TextPaint ds) {
            super.updateDrawState(ds);
            ds.setUnderlineText(false);
        }
    }

}
       {% endhighlight %}
       
效果如下：

//======================

这样textview在界面上就不会有下划线了，但却保留着autolink的功能。
这个超链接是默认颜色，如果需要改变颜色可以在xml中设置android:textColorLink="#1e84fb"，
或者在java代码中设置tv.setLinkTextColor(color);

上面的例子是在autoLink设置的情况下自动识别超链接的，如果不需要自动识别，而是自己手动设置需要跳转的网址，
可以使用以下方法，首先删除xml中的android:autoLink="email|phone|web"

      {% highlight java %} 
public class Test10Activity extends Activity {

    TextView textView ;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test10);
        textView = (TextView) findViewById(R.id.text);

        String webLinkText = "我的博客" ;
        SpannableString text = new SpannableString(webLinkText);
        NoUnderlineSpan mNoUnderlineSpan = new NoUnderlineSpan("https://souly.cn") ;
        text.setSpan(mNoUnderlineSpan,0,text.length(),Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
        textView.setText(text);
        textView.setMovementMethod(LinkMovementMethod.getInstance());
    }

    public static class NoUnderlineSpan extends URLSpan {
        public NoUnderlineSpan(String url) {
            super(url);
        }
        @Override
        public void updateDrawState(TextPaint ds) {
            super.updateDrawState(ds);
            ds.setUnderlineText(false);
        }
    }

}
       {% endhighlight %}
       


