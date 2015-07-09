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




然后在，项目的build.gradle中进行如下配置：

    {% highlight java  %}

apply plugin: 'com.android.application'
// 1.test plugin
apply plugin: 'org.robolectric'
android {
    compileSdkVersion 22
    buildToolsVersion "22.0.1"

    defaultConfig {
        applicationId "kale.testapplication"
        minSdkVersion 15
        targetSdkVersion 18
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    packagingOptions {
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/NOTICE'
        exclude 'META-INF/NOTICE.txt'
        exclude 'LICENSE.txt'
    }
}

dependencies {
    androidTestCompile 'junit:junit:4.12'
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.1.1'
    // 2. test-libs
    androidTestCompile 'org.robolectric:robolectric:2.4'
}

// 3. test settings
robolectric {
    // Configure includes / excludes
    include '**/*Test.class'
    exclude '**/espresso/**/*.class'

    // Configure max heap size of the test JVM
    maxHeapSize = '2048m'

    // Configure the test JVM arguments - Does not apply to Java 8
    jvmArgs '-XX:MaxPermSize=512m', '-XX:-UseSplitVerifier'

    // Specify max number of processes (default is 1)
    maxParallelForks = 4

    // Specify max number of test classes to execute in a test process
    // before restarting the process (default is unlimited)
    forkEvery = 150

    // configure whether failing tests should fail the build
    ignoreFailures true

    // use afterTest to listen to the test execution results
    afterTest { descriptor, result ->
        println "Executing test for ${descriptor.name} with result: ${result.resultType}"
    }
}



    {% endhighlight %}

