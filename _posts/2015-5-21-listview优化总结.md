---
layout: post
author: mxn
title: listview优化总结
category: 技术博文
tag: [android]
---

1、ViewHolder,Tag 必不可少，viewHolder要使用static关键字修饰，防止内存泄漏。

2、如果自定义Item中有涉及到图片等等的，一定要处理图片，图片占的内存是ListView项中最多的，处理图片的方法大致有以下几种：
2.1：不要直接拿个路径就去循环decodeFile();用Option保存图片大小、不要加载图片到内存去;
2.2:  拿到的图片一定要经过边界压缩
2.3:在ListView中取图片时也不要直接拿个路径去取图片，而是以WeakReference（使用WeakReference代替强引用。比如可以使用WeakReference<Context> mContextRef）、SoftReference、WeakHashMap等的来存储图片信息，不是图片。
2.4:在getView中做图片转换时，产生的中间变量一定及时释放.

3、尽量避免在BaseAdapter中使用static 来定义全局静态变量，static是Java中的一个关键字，当用它来修饰成员变量时，那么该变量就属于该类，而不是该类的实例。所以用static修饰的变量，它的生命周期是很长的，如果用它来引用一些资源耗费过多的实例（比如Context的情况最多），这时就要尽量避免使用了.

4、如果为了满足需求下必须使用Context的话：Context尽量使用Application Context，因为Application的Context的生命周期比较长，引用它不会出现内存泄露的问题

5、尽量避免在ListView适配器中使用线程，因为线程产生内存泄露的主要原因在于线程生命周期的不可控制
