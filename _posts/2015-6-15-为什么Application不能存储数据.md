---
layout: post
author: mxn
title: 为什么Application不能存储数据
category: 技术博文
tag: [gradle]
---

###不要在Application中存储数据

有一些数据可能会在app的很多地方使用到，也许是一些 session token，或者计算结果，等等。我们会偷懒不想把这些数据在activity之间不停地传递，或者保存在硬盘上。
所以有人为了让数据可以简单地跨越activity使用，就将数据存储在Application中，这个解决方案看上去简单，优雅，但是缺是完全错误的。
