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

  <!-- more -->


然后在，项目的build.gradle中进行如下配置：

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

