---
layout: post
author: mxn
titile: android ART学习
category: 技术博文
tag: android
---

Android L默认采用ART运行环境，完全兼容64位移动处理器。Google称这将比此前的Dalvik模式性能提高两倍，但是会占用更多的内存空间。Android有三种运行模式：Native、Dalvik、ART。Dalvik明显是最慢的，完全的Native模式是最快的但是开发难度高，ART介于两者之间，并且不影响现有开发模式，所以Google选了ART。

#### Native:

在开发过程中使用编译器在C/C++等语言直接编译成机器码，运行的时候能够充分利用系统性能，这是最快的。iOS的Object C和Android 的NDK都是这种模式。

#### Dalvik:

Android L系统之前所有Android版本的运行方式，采用的是字节码，在运行的时候解释执行变成机器能够识别的机器码。这个过程是比较缓慢的。

#### ART :

Android 4.4开始推出的新的运行环境，在APP安装的时候使用dex2oat工具直接把DEX文件转换为机器码文件，运行的时候以机器码方式运行，可以充分利用系统性能；此外，改进的内存回收机制使得ART运行模式下的内存回收速度只有Dalvik运行时模式下的50%，也能够提升系统运行速度。

![](http://a.36krcnd.com/photo/2014/a7eaa64191b5b05b1bb8702dfab7a735.png)

虽然运行的速度提高了，但是ART也有缺点：

1）APP安装过程会变慢；

2）APP占用的存储空间会变多，系统更容易出现系统空间不足问题。
