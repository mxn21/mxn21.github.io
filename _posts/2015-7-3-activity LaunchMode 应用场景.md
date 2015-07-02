---
layout: post
author: mxn
titile: activity LaunchMode 应用场景
category: 技术博文
tag: android
---

Activity的启动模式一种有四种，分别如下：

1、standard

2、singleTop

3、singleTask

4、singleInstance


standard：不论当前任务栈中是否存在该Activity，都会新建一个Activity，如 任务栈为A B，要启动B 那么任务栈为 A B B

singleTop：如果当前要创建的Activity就在任务栈的顶端，那么不会创建新的Activity，仅仅调用Activity的onNewIntent，如果不在栈顶(或者栈中没有该Activity)，那么还是会创建新的Activity，如任务栈为A B 启动B 任务栈变为 A B 如果 启动A 那么任务栈为 A B A

singleTask：如果当前任务中存在要启动的Activity，那么就不会创建新的Activity，如果不存在就会创建新的Activity，如任务栈为 A B C，启动B ，那么任务栈就会变为A B

singleInstance：将一个Activity的launchMode设置为该值时，表明这个Activity独自占用一个任务队列，这个队列中不让在加入其他的Activity


