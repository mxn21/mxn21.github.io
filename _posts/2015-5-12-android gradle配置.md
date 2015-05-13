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

{% highlight c %}
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

<!-- more -->

#####4.编译android项目的参数

{% highlight c %}
android {
        compileSdkVersion 22
        buildToolsVersion "22.0.1"
     
        defaultConfig {
            minSdkVersion 16
            targetSdkVersion 20
        }
        //Android默认配置
        sourceSets {
            main {
                manifest.srcFile 'AndroidManifest.xml'
                java.srcDirs = ['src']
                resources.srcDirs = ['src']
                aidl.srcDirs = ['src']
                renderscript.srcDirs = ['src']
                res.srcDirs = ['res']
                assets.srcDirs = ['assets']
            }
            //测试所在的路径，这里假设是tests文件夹，没有可以不写这一行
            instrumentTest.setRoot('tests')
        }
         
        //这个是解决lint报错的代码
        lintOptions {
            abortOnError false
        }
        /**
         * 签名设置
         */
        signingConfigs {
            release {
            def filePrivateProperties = file("private.properties")
            if (filePrivateProperties.exists()) {
                Properties propsPrivate = new Properties()
                propsPrivate.load(new FileInputStream(filePrivateProperties))
                storeFile file(propsPrivate['key.store'])
                keyAlias propsPrivate['key.alias']
                storePassword propsPrivate['key.store.password']
                keyPassword propsPrivate['key.alias.password']
            }
        }
        }
        /**
         * 混淆设置
         */
        buildTypes {
            release {
                signingConfig signingConfigs.myConfigs
                runProguard true
                proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            }
        }
        /**
         * 渠道打包（不同包名）
         */
        productFlavors {
        // for development
            dev {
          applicationId = "包名"
          }
           // for Google Play Store release
            play {
            applicationId "包名"
         }
        
        }
    }
{% endhighlight %}


#####5.本地依赖

{% highlight c %}
dependencies { 
        //单文件依赖
    compile files('libs/android-support-v4.jar')   
    //某个文件夹下面全部依赖
    compile fileTree(dir: 'libs', include: '*.jar')
}
 
android {
     
}
{% endhighlight %}

#####6.远程依赖

{% highlight c %}
repositories { 
        //从中央库里面获取依赖
    mavenCentral()  
    //或者使用指定的本地maven 库
    maven{
        url "file://F:/githubrepo/releases"
    }   
    //或者使用指定的远程maven库
    maven{
        url "远程库地址"
    }
}
 
dependencies { 
        //应用格式: packageName:artifactId:version
    compile 'com.google.android:support-v4:r13'}
 
android {
 
}

{% endhighlight %}
