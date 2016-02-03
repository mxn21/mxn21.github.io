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
       
这里的NoUnderlineSpan继承了URLSpan而不是UnderlineSpan，URLSpan是ClickableSpan的子类。

效果图如下：

//======================

### Linkify.addLinks方式

除了使用默认的web等模式之外，我们还可以通过Linkify类的addLinks方法来添加自定义模式。Linkify.addLinks也是功能最
强大的一种方式。

在讲这种方式之前先科普一下Intent Filters的知识。

android中的activity,service,Broadcast Receivers都可以设置intent过滤器，它们可以有一个或多个intent过滤器。
每个过滤器描述组件的一种能力，即告知系统能够处理哪些隐式intent。个显式intent总是能够传递到它的目标组件，不管它包含什么；
不考虑过滤器。但是一个隐式intent，仅当它能够通过组件的过滤器之一才能够传递给它。

一个intent过滤器是一个IntentFilter类的实例。因为Android系统在启动一个组件之前必须知道它的能力，但是intent过滤器通常不在java
代码中设置，而是在应用程序的清单文件（AndroidManifest.xml）中以<intent-filter>元素设置。但有一个例外，
广播接收者的过滤器通过调用Context.registerReceiver()动态地注册，它直接创建一个IntentFilter对象。

一个过滤器有对应于Intent对象的动作、数据、种类的字段。过滤器要检测隐式intent的所有这三个字段，其中任何一个失败，
Android系统都不会传递intent给组件。然而，因为一个组件可以有多个intent过滤器，一个intent通不过组件的过滤器检测，
其它的过滤器可能通过检测。

*动作检测

清单文件中的<intent-filter>元素以<action>子元素列出动作，例如：

      {% highlight xml %} 
<intent-filter>
    <action android:name="com.example.project.SHOW_CURRENT" />
    <action android:name="com.example.project.SHOW_RECENT" />
    <action android:name="com.example.project.SHOW_PENDING" />
    . . .
</intent-filter>
     {% endhighlight %}
     
像例子所展示，虽然一个Intent对象仅是单个动作，但是一个过滤器可以列出不止一个。这个列表不能够为空，一个过滤器必须至少包含一个
<action>子元素，否则它将阻塞所有的intents。

要通过检测，Intent对象中指定的动作必须匹配过滤器的动作列表中的一个。如果对象或过滤器没有指定一个动作，结果将如下：

如果过滤器没有指定动作，没有一个Intent将匹配，所有的intent将检测失败，即没有intent能够通过过滤器。
如果Intent对象没有指定动作，将自动通过检查（只要过滤器至少有一个<action>子元素，否则就是上面的情况了）


*种类检测

类似的，清单文件中的<intent-filter>元素以<category>子元素列出种类，例如：

      {% highlight xml %} 
      
<intent-filter>
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
    . . .
</intent-filter>

     {% endhighlight %}
     
对于一个intent要通过种类检测，intent对象中的每个种类必须匹配过滤器中的一个。即过滤器能够列出额外的种类，但是intent对象中的种类都必须能够在过滤器中找到，
只要一个种类在过滤器列表中没有，就算种类检测失败！

因此，原则上如果一个intent对象中没有种类（即种类字段为空）应该总是通过种类测试，而不管过滤器中有什么种类。但是有个例外，
Android对待所有传递给Context.startActivity()的隐式intent好像它们至少包含"android.intent.category.DEFAULT"
（对应CATEGORY_DEFAULT常量）。因此，activity想要接收隐式intent必须要在intent过滤器中包含"android.intent.category.DEFAULT"。

注意："android.intent.action.MAIN" 和 "android.intent.category.LAUNCHER"设置，它们分别标记活动开始新的任务和带到启动列表界面。
它们可以包含"android.intent.category.DEFAULT"到种类列表，也可以不包含。

*数据检测

类似的，清单文件中的<intent-filter>元素以<data>子元素列出数据，例如：

      {% highlight xml %} 
<intent-filter>
    <data android:mimeType="video/mpeg" android:scheme="http" /> 
    <data android:mimeType="audio/mpeg" android:scheme="http" />
    . . .
</intent-filter>
     {% endhighlight %}

每个<data>元素指定一个URI和数据类型（MIME类型）。它有几个属性scheme、host、port、path、query、fragment对应于URI的每个部分： 
scheme://host:port/path?query#fragment或者scheme://userInfo@host:port/path?query#fragment。

scheme是content，host是"com.example.project"，port是200，path是"folder/subfolder/etc"。host和port一起构成URI的凭据
（authority），如果host没有指定，port也被忽略。 这几个属性都是可选的，但它们之间并不都是完全独立的。要让authority有意义，
scheme必须也要指定。要让path有意义，scheme和authority也都必须要指定。

当比较intent对象和过滤器的URI时，仅仅比较过滤器中出现的URI属性。例如，如果一个过滤器仅指定了scheme，所有有此scheme的URIs都匹配过滤器；
如果一个过滤器指定了scheme和authority，但没有指定path，所有匹配scheme和authority的URIs都通过检测，而不管它们的path；
如果几个属性都指定了，要都匹配才能算是匹配。然而，过滤器中的path可以包含通配符来要求匹配path中的一部分。

<data>元素的type属性指定数据的MIME类型。Intent对象和过滤器都可以用"*"通配符匹配子类型字段，例如"text/*"，"audio/*"表示任何子类型。

数据检测既要检测URI，也要检测数据类型。规则如下：

1.一个Intent对象既不包含URI，也不包含数据类型：仅当过滤器也不指定任何URIs和数据类型时，才不能通过检测；否则都能通过。
2.一个Intent对象包含URI，但不包含数据类型：仅当过滤器也不指定数据类型，同时它们的URI匹配，才能通过检测。例如，mailto:和tel:都不指定实际数据。
3.一个Intent对象包含数据类型，但不包含URI：仅当过滤也只包含数据类型且与Intent相同，才通过检测。
4.一个Intent对象既包含URI，也包含数据类型（或数据类型能够从URI推断出）：数据类型部分，只有与过滤器中之一匹配才算通过；URI部分，它的URI要出现在过滤器中，或者它有content
:或file: URI，又或者过滤器没有指定URI。换句话说，如果它的过滤器仅列出了数据类型，组件假定支持content:和file: 。
如果一个Intent能够通过不止一个活动或服务的过滤器，用户可能会被问那个组件被激活。如果没有目标找到，会产生一个异常。


下面写一个例子来通过定义Scheme接收特定URI开启Activity

首先在AndroidManifast.xml要给被指定Scheme的Activity下设置如下参数

      {% highlight xml %} 
        <intent-filter>  
                <category android:name="android.intent.category.DEFAULT"></category>  
                <action android:name="android.intent.action.VIEW"></action>  
                <data android:host="profile" android:scheme="mxn"/>
        </intent-filter>  
      {% endhighlight %}
      
这样即指定了接收Uri的Scheme为“mxn”,host为“profile” 且 Action为View的Intent。
可以利用如下Intent调用Activity：  startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("mxn://profile?uid=1"))); 
传递一个参数uid=1. 

在设置了filter的activity中接受参数：

      {% highlight java %} 
public class Test9Activity extends Activity {

    private String uid;

    private static final Uri PROFILE_URI = Uri.parse("mxn://profile");

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test8);
        extractUidFromUri();

    }
    private void extractUidFromUri() {
        Uri uri = getIntent().getData();
        if (uri != null && PROFILE_URI.getScheme().equals(uri.getScheme())) {
            uid = uri.getQueryParameter("uid");
            Log.d("=====", "uid from url: " + uid);
        }
    }
}
      {% endhighlight %}
      
下面借助Linkify实现超链接，实现微博中@的超链接功能，布局文件如下：

      {% highlight xml %} 
      <RelativeLayout
          xmlns:android="http://schemas.android.com/apk/res/android"
          xmlns:tools="http://schemas.android.com/tools"
          android:layout_width="match_parent"
          android:layout_height="match_parent"
          android:gravity="center"
          >    
             <TextView
                 android:id="@+id/text"
                 android:layout_width="wrap_content"
                 android:layout_height="wrap_content"
                 android:textSize="20sp"
                 android:text="【谷歌母公司Alphabet市值超苹果：全球第一】据外媒报道，@谷歌 母公司 @Alphabet 
                 通过公司分拆以及搜索广告业务的持续强势，股价继续攀升。美国时间周一，Alphabet的公司市值已超过了 @苹果 ，
                 成为全世界市值最大的公司. www.google.com"
                 />
      </RelativeLayout>
      {% endhighlight %}
      
当需要使自定义模式和内置模式web，phone等一起被识别时，一定要先声明内置模式，然后再声明自定义模式，而且不能在xml中通过autoLink属性声明，
否则自定义模式不起作用。因为在设置内置模式时，会先删除已有模式。
下面使用正则匹配带“@”的用户名，然后作为uid传递到下一个页面：

      {% highlight java %} 
public class Test10Activity extends Activity {

    TextView textView ;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test10);
        textView = (TextView) findViewById(R.id.text);
        Pattern p = Pattern.compile("@(\\w+?)(?=\\W|$)(.)");
        Linkify.addLinks(textView, p, "mxn://profile?uid=");
    }
}
      {% endhighlight %}

接收的时候跟上面一样：

      {% highlight java %} 
public class Test9Activity extends Activity {

    private String uid;

    private static final Uri PROFILE_URI = Uri.parse("mxn://profile");

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test8);
        extractUidFromUri();

    }
    private void extractUidFromUri() {
        Uri uri = getIntent().getData();
        if (uri != null && PROFILE_URI.getScheme().equals(uri.getScheme())) {
            uid = uri.getQueryParameter("uid");
            Log.d("=====", "uid from url: " + uid);
        }
    }
}

      {% endhighlight %}
      
效果如下：

//==================

其实在上面的图上可以看到，www.google.com并没有被识别为链接（因为我们没有设置web模式）。需要注意的是，当需要使自定义
模式和内置模式web，phone等一起被识别时，一定要先声明内置模式，然后再声明自定义模式，而且不能在xml中通过autoLink属性声明，
否则自定义模式不起作用。因为在设置内置模式时，会先删除已有模式。

将上面的Java代码改为：

      {% highlight java %} 
public class Test10Activity extends Activity {

    TextView textView ;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test10);
        textView = (TextView) findViewById(R.id.text);
        Pattern p = Pattern.compile("@(\\w+?)(?=\\W|$)(.)");
        Linkify.addLinks(textView, Linkify.WEB_URLS);
        Linkify.addLinks(textView, p, "mxn://profile?uid=");
    }
}     
      {% endhighlight %}

现在就可以同时识别web以及自定义模式了。

//==================

同样的，这个超链接是默认颜色，如果需要改变颜色可以在xml中设置android:textColorLink="#1e84fb"，或者在java代码中
设置tv.setLinkTextColor(color);

这时超链接还是
