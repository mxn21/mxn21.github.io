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


首先，在project最外面的build.gradle中配置classpath

    {% highlight java  %}

buildscript {
    repositories {
        jcenter()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.1.0'
        classpath 'org.robolectric:robolectric-gradle-plugin:1.0.1'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
        mavenCentral()
    }
}


    {% endhighlight %}


    {% highlight java  %}

    dependencies {
       testCompile "org.robolectric:robolectric:2.4"
    }

    {% endhighlight %}

  <!-- more -->