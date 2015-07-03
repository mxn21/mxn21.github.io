---
layout: post
author: mxn
titile: activity LaunchMode 应用场景
category: 技术博文
tag: android
---

在看LaunchMode之前先理解一下Task。task是一个stack（栈），"Last in, First out"。一个task包含了一组activity实例。
一般情况下，当用户打开一个新的app的时候一个新的task就被创建了，第一个打开的activity被叫做task的root。安卓系统可以同时运行很多个
task，但是只有一个task在前台。

通过LaunchMode可以设置不同的activity关联到task的方式。

Activity的启动模式一种有四种，分别如下：

1、standard

2、singleTop

3、singleTask

4、singleInstance

这四种类型可以分为两大类：
'standard' 和 'singleTop'可以在同一个栈中加载很多activity。区别于下面一组在于，这两个模式不会移除其他Activity。

'singleTask' 和 'singleInstance'的Activity会使用单例模式，并且这个Activity会是一个新的栈的root。

launch mode可以在 AndroidManifest文件中配置，也可以使用Intent flags，比如FLAG_ACTIVITY_NEW_TASK,
FLAG_ACTIVITY_CLEAR_TOP 和 FLAG_ACTIVITY_SINGLE_TOP 。



standard：不论当前任务栈中是否存在该Activity，都会新建一个Activity，如 任务栈为A B，要启动B 那么任务栈为 A B B

singleTop：如果当前要创建的Activity就在任务栈的顶端，那么不会创建新的Activity，仅仅调用Activity的onNewIntent，如果不在栈顶(或者栈中没有该Activity)，那么还是会创建新的Activity，如任务栈为A B 启动B 任务栈变为 A B 如果 启动A 那么任务栈为 A B A

singleTask：如果当前任务中存在要启动的Activity，那么就不会创建新的Activity，如果不存在就会创建新的Activity，如任务栈为 A B C，启动B ，那么任务栈就会变为A B

singleInstance：将一个Activity的launchMode设置为该值时，表明这个Activity独自占用一个任务队列，这个队列中不让在加入其他的Activity




### 应用场景

#### SingleTop

应用场景：网易新闻。
假设主界面为 MainActivity，显示新闻的界面是 DetailActivity，显然显示任何一条新闻都会使用 DetailActivity，即把新闻内容通过 Intent 传给 DetailActivity 就可以了。
假设你正在看新闻1(即在 DetailActivity)，此时手机收到服务器的推送：收到一条通知(新闻2)，点击通知就会跳转到 DetailActivity 并显示新闻2，当你点击通知时，因为目前栈顶的 Activity 就是 DetailActivity，因此这里就是使用 SingleTop 的地方，即点击通知后以 SingleTop 加载模式打开 DetailActivity 并显示新闻2，因此新闻1的 DetailActivity 就被覆盖掉了。
此后你点击返回键会回到主界面。

#### SingleTask

应用场景：微信的主界面(一般应用主界面都会以 SingleTask 启动)。
你打开微信主界面(在栈的最底部)后，进入朋友圈(在栈的顶部)，此时你点击 Home 键回桌面，并打开网易新闻。
假设你想将网易新闻的一条新闻分享给微信好友，那么就按照 分享->微信->好友A->分享给他->留在微信。接着会跳转微信的主界面，即不是你原本所在的朋友圈，并且微信的栈只剩下一个元素：主界面的 Activity。这里就使用了 SingleTask(即以 SingleTask 加载模式打开微信主界面)。

#### SingleInstance

应用场景：闹铃的响铃界面。
你以前设置了一个闹铃：上午6点。在上午5点58分，你启动了闹铃设置界面，并按 Home 键回桌面；在上午5点59分时，你在微信和朋友聊天；在6点时，闹铃响了，并且弹出了一个对话框形式的 Activity(名为 AlarmAlertActivity) 提示你到6点了(这个 Activity 就是以 SingleInstance 加载模式打开的)，你按返回键，回到的是微信的聊天界面，这是因为 AlarmAlertActivity 所在的 Task 的栈只有他一个元素，因此退出之后这个 Task 的栈空了。如果是以 SingleTask 打开 AlarmAlertActivity，那么当闹铃响了的时候，按返回键应该进入闹铃设置界面。