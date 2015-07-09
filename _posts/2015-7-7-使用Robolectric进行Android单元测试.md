---
layout: post
author: mxn
titile: 使用Robolectric进行Android单元测试
category: 技术博文
tag: android
---

### 测试驱动android开发

在安卓模拟器或者真机上跑测试用例速度很慢。构建、部署、启动app，通常需要花费一分钟或者更久。这不是TDD（测试驱动开发）模式.Robolectric提供一种更好的方式。

可能你一直尝试在java IDE中使用junit或者testng直接跑测试用例，但是一直报java.lang.RuntimeException: Stub!异常。

这个异常是因为在jdk中没有android运行环境。而现在Robolectric这款android单元测试工具，模拟了android sdk中的jar包，可以直接在jvm中运行测试用例，这样就大大节省了时间。

### SDK，Resource和Native Method模拟

Robolectric可以处理控件展示、资源加载和很多使用native C实现的真机上的其他功能。所以我们可以使用Robolectric来模拟真机上的大部分操作。我们可以很方便地获取Robolectric的源码，直接查看它的模拟机制，所以使用Robolectric，我们也可以模拟错误条件和一些真实的传感器信号。

### 脱离模拟器执行测试用例

Robolectric允许我们在项目工程中或者持续集成（CI如hudson、jenkins）中使用jvm来执行测试用例，所以就省掉了打包、安装的过程，将测试用例的执行时间由分钟级降到秒级。

### 配置

#### 准备测试

在Robolectric测试之前，有一些事情需要准备：
1、建立一个测试文件夹，使Gradle和 Android Studio都能识别它
2、在project中加入Gradle Android Test Plug
3、加入自定义TestRunner

  <!-- more -->


#### 建立测试文件夹

Android Studio新建项目的时候并没有建立测试文件夹，如果使用Eclipse需要新建一个项目用来测试。
因为Android Studio的项目基于Gradle构建，所以只要在项目下建立测试文件夹就可以了。
我们可以把测试文件放在 src/test下，为了保证测试可以运行，我们需要告诉Gradle 和 Android Studio相对路径 。

1、创建src/test/java用了放测试文件
2、编辑build.gradle，在Android标签下，与defaultConfig同一级的地方加入配置

    {% highlight java  %}
    sourceSets {
        instrumentTest.setRoot('src/test')
     }

    {% endhighlight %}

#### 加入Gradle Test Plug-in

通过Gradle运行Android unit tests需要加入 Gradle Android Test plug-in

1、配置maven如下

    {% highlight java  %}
   maven {
      url 'https://oss.sonatype.org/content/repositories/snapshots/'
  }
    {% endhighlight %}

2、在build.gradle中的dependencies加入plug-in
 classpath 'com.squareup.gradle:gradle-android-test-plugin:0.9.1-SNAPSHOT'

3、加入  apply plugin: 'android-test'

4、加入 test-only dependencies:

 testCompile 'junit:junit:4.10'
 testCompile 'org.robolectric:robolectric:2.1.+'
 testCompile 'com.squareup:fest-android:1.0.+'
 instrumentTestCompile 'junit:junit:4.10'
 instrumentTestCompile 'org.robolectric:robolectric:2.3-SNAPSHOT'
 instrumentTestCompile 'com.squareup:fest-android:1.0.+'

最终的build.gradle文件如下

    {% highlight java  %}
    buildscript {
      repositories {
          mavenCentral()
          maven {
              url 'https://oss.sonatype.org/content/repositories/snapshots/'
          }
      }
      dependencies {
          classpath 'com.android.tools.build:gradle:0.6.+'
          classpath 'com.squareup.gradle:gradle-android-test-plugin:0.9.1-SNAPSHOT'
      }
    }
    apply plugin: 'android'
    apply plugin: 'android-test'

    repositories {
      mavenCentral()
      maven {
          url 'https://oss.sonatype.org/content/repositories/snapshots/'
      }
    }

    android {
      compileSdkVersion 18
      buildToolsVersion "18.1.0"

      defaultConfig {
          minSdkVersion 7
          targetSdkVersion 18
      }

      sourceSets {
          instrumentTest.setRoot('src/test')
      }
    }

    dependencies {
      compile 'com.android.support:appcompat-v7:+'

      testCompile 'junit:junit:4.10'
      testCompile 'org.robolectric:robolectric:2.3-SNAPSHOT'
      testCompile 'com.squareup:fest-android:1.0.+'
      instrumentTestCompile 'junit:junit:4.10'
      instrumentTestCompile 'org.robolectric:robolectric:2.3-SNAPSHOT'
      instrumentTestCompile 'com.squareup:fest-android:1.0.+'

    }
    {% endhighlight %}

#### 加入自定义Test Runner

Robolectric会在测试启动阶段读取AndroidManifest.xml，如果测试在src/test/java下则无法找到AndroidManifest.xml，
我们需要加入自定义Test Runner来解决这个问题。

在test文件夹下加入文件，如下：

    {% highlight java  %}
    import android.app.Fragment;
    import android.app.FragmentManager;
    import android.app.FragmentTransaction;
    import android.support.v4.app.FragmentActivity;

    import org.junit.runners.model.InitializationError;
    import org.robolectric.AndroidManifest;
    import org.robolectric.Robolectric;
    import org.robolectric.RobolectricTestRunner;
    import org.robolectric.annotation.Config;
    import org.robolectric.res.Fs;

    public class RobolectricGradleTestRunner extends RobolectricTestRunner {

      public RobolectricGradleTestRunner(Class<?> testClass) throws InitializationError {
       super(testClass);
      }

     @Override
    protected AndroidManifest getAppManifest(Config config) {
          String manifestProperty = System.getProperty("android.manifest");
          if (config.manifest().equals(Config.DEFAULT) && manifestProperty != null) {
             String resProperty = System.getProperty("android.resources");
            String assetsProperty = System.getProperty("android.assets");
           return new AndroidManifest(Fs.fileFromPath(manifestProperty), Fs.fileFromPath(resProperty),
                      Fs.fileFromPath(assetsProperty));
          }
          AndroidManifest appManifest = super.getAppManifest(config);
          return appManifest;
     }

    }
        {% endhighlight %}

#### 写一个Robolectric Test

在src/test/java下写一个Robolectric Test

    {% highlight java  %}

    package de.peterfriese.robolectricdemo;

    import org.junit.Test;
    import org.junit.runner.RunWith;

    import de.peterfriese.robolectric.RobolectricGradleTestRunner;

    import static org.junit.Assert.assertTrue;

    @RunWith(RobolectricGradleTestRunner.class)
    public class MainActivityTest {

      @Test
     public void shouldFail() {
         assertTrue(false);
     }
    }
    {% endhighlight %}

有一些注意事项：需要使用刚创建的RobolectricGradleTestRunner，这样测试才能找到manifest。
我们加入了一个简单的测试方法assertTrue(false);这个断言是注定失败的测试方法，如果运行的时候出现test fails，则说明运行正常。

下面运行测试

在Build Variants view中切换到unit tests
![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img24.png)

这样我们的测试文件夹会变成绿色

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img25.png)

输入测试命令

./gradlew test

./gradlew check

./gradlew build

test会直接运行tests, check 会运行tests和checkstyle等等, build会compiles包括test和check的一切。


    {% highlight c  %}
$ ./gradlew test

de.peterfriese.robolectricdemo.MainActivityTest > shouldFail FAILED
java.lang.AssertionError at MainActivityTest.java:15

1 test completed, 1 failed
:RobolectricDemo:testDebug FAILED

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':RobolectricDemo:testDebug'.
> There were failing tests. See the report at: file:///Users/peterfriese/Projects/peterfriese.de/Robolectric/RobolectricDemoProject/RobolectricDemo/build/test-report/debug/index.html

* Try:
Run with --stacktrace option to get the stack trace. Run with --info or --debug option to get more log output.

BUILD FAILED

    {% endhighlight %}

和预想的一样出现test failed，可以在浏览器中打开../RobolectricDemoProject/RobolectricDemo/build/test-report/debug/index
.html来查看更多信息。

#### 写另一个Robolectric Test

    {% highlight java  %}
@RunWith(MyTestRunner.class)
public class SignInScreenTest {

    @Test
    public void should_start_intent_when_click_registration_button() {
    2   Activity activity = new Activity();
        SignInScreen signInScreen = new SignInSceen(activity);
    3   TextView textView = (TextView)  signInScreen.findViewById(R.id.sign_in_registration);
         textView.performClick();

    4   ShadowActivity shadowActivity = Robolectric.shadowOf(activity);
        Intent nextStartedActivity = shadowActivity.getNextStartedActivity();
        ShadowIntent shadowIntent = Robolectric.shadowOf(nextStartedActivity);
        assertThat((Class<WebPageActivity>) shadowIntent.getIntentClass(), equalTo(WebPageActivity.class));
    }
}
    {% endhighlight %}

在这段测试代码中：

（1）声明了测试运行的test runner；就像普通的单元测试，它也分为了set up， method invoke，以及assert三个阶段。
在（2）中，测试初始化了一个Activity用于提供Context，并使用这个Activity对象生成了一个SignInScreen实例；
第二个阶段，也是就（3）中，代码在生成的登录界面中找到注册按钮，并进行点击。最为有意思的第三个阶段需要验证注册按钮的点击触发了我们期望的事件，即使用Implicit Intent来打开WebPageActivity。
为了进行这个验证，（4）中首先通过Robolectric的静态方法shadowOf来获取activity对象相应的Shadow Object ，而通过这个Shadow Object,代码获得了activity对象的所开启的Intent对象。最后通过Intent对象的Shadow Object ，我们可以获得其intent class并进行验证。
通过这个测试我们可以看到，有了Robolectric的帮助，我们可以轻松的生成Activity实例，加载xml布局文件，进行组件上的方法调用。通过shadow对象，我们则可以获取Android相关类的对象状态信息，来对测试的结果进行验证。实际上除了Intent，我们还可以对通过使用Robolectric对代码中的Dialog，HTTP请求，数据库操作等各个方面进行测试。

Robolectric并没有为Android SDK中的所有类都定义shadow对象，你可以通过调用 Robolectric.getDefaultShadowClasses() 方法来查看你所需要的类是否已经被注册到了需要被shadow的类列表中。如果没有你可能就需要对其进行定制和扩展。关于如何添加Shadow Objects而增加Robolectric的功能，在Robolectric的网站文档中有详细的描述。

由于Robolectric的测试是可以脱离Android的SDK运行于JVM上，我们就可以像运行普通的jUnit测试一样在IDE中或者在终端使用构建脚本运行我们的测试。


#### 测试用户交互

确保静态导入Android FEST assertations

import static org.fest.assertions.api.ANDROID.assertThat;

我们需要在tests获取activity

    {% highlight java  %}

    private MainActivity activity;

    @Before
    public void setup() {
      activity = Robolectric.buildActivity(MainActivity.class).get();
    }
    {% endhighlight %}

验所有UI elements被初始化

    {% highlight java  %}
@Test
public void shouldNotBeNull() {
  assertThat(activity).isNotNull();

  TextView textView = (TextView) activity.findViewById(R.id.textView);
  assertThat(textView).isNotNull();

  Button button = (Button) activity.findViewById(R.id.button);
  assertThat(button).isNotNull();

  EditText editText = (EditText) activity.findViewById(R.id.editText);
  assertThat(editText).isNotNull();
}
    {% endhighlight %}

下面写一个测试，在edit中放入一些文字，点击按钮后与我们规定的内容比较

    {% highlight java  %}
@Test
public void shouldProduceGreetingWhenButtonPressed() {
  TextView textView = (TextView) activity.findViewById(R.id.textView);
  Button button = (Button) activity.findViewById(R.id.button);
  EditText editText = (EditText) activity.findViewById(R.id.editText);

  editText.setText("Peter");
  button.performClick();

  assertThat(textView).containsText("Hello, Peter!");
}

    {% endhighlight %}

当运行的时候，测试会fail。需要按下面的方法修复。
打开MainActivity.java。加入如下的private class。

    {% highlight java  %}
public static class MainFragment extends Fragment {
  @Override
  public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
      final View rootView = inflater.inflate(R.layout.fragment_main, container, false);
      Button button = (Button) rootView.findViewById(R.id.button);
      button.setOnClickListener(new View.OnClickListener() {
          @Override
          public void onClick(View view) {
              TextView textView = (TextView) rootView.findViewById(R.id.textView);
              EditText editText = (EditText) rootView.findViewById(R.id.editText);

              textView.setText(String.format("Hello, %s!", editText.getText()));
          }
      });

      return rootView;
  }
}
    {% endhighlight %}

在代码中设置了OnClickListener监听，然后把editText的内容给了textView。
同属需要改变MainActivity的构造函数，这样就可以在初始化的时候使用新的fragment class

    {% highlight java  %}
     if (savedInstanceState == null) {
          getSupportFragmentManager().beginTransaction()
                  .add(R.id.container, new MainFragment())
                  .commit();
      }
     {% endhighlight %}

现在测试通过了

    {% highlight c  %}
    $ ./gradlew test
    BUILD SUCCESSFUL

    Total time: 7.066 secs
     {% endhighlight %}