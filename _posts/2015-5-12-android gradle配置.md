---
layout: post
author: mxn
title: android gradle配置
category: 技术博文
tag: [gradle]
---


#####1.去除无用的资源：

{% highlight c %}
android {
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
        }
    }
}
{% endhighlight %}


#####2.多渠道打包修改如下:

{% highlight c %}

productFlavors {
        demo {
            applicationId "com.example.app.demo"
            versionName "1.0-demo"
        }
        full {
            applicationId "com.example.app.full"
            versionName "1.0-full"
        }
    }


{% endhighlight %}

#####3.脚本的运行环境:

{% highlight html %}
 buildscript {
         //支持java 依赖库管理（maven/ivy）,用于项目的依赖。
     repositories {
            mavenCentral()
        }
        //依赖包的定义。支持maven/ivy，远程，本地库，也支持单文件
        dependencies {
            classpath 'com.android.tools.build:gradle:1.2.2'
        }
    }

{% endhighlight %}

