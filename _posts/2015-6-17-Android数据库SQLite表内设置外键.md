---
layout: post
author: mxn
title: Android数据库SQLite表内设置外键
category: 技术博文
tag: [android]
---

####介绍

Android默认的数据是SQLite，但SQLite3.6.19之前(在2.2版本中使用的是3.6.22，因此如果你的应用只兼容到2.2版本就可以放心使用外键功能)是不支持外键的，如果有两张表需要关联，用外键是最省事的，但不支持的话怎么办呢？这里就有一个解决办法，就是用事务将两张表关联起来，并且最后生成一张视图。

####现有两张表
Employees
Dept
####视图
![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img2.JPG)

ViewEmps:显示雇员信息和他所在的部门.

####创建数据库
自定义一个辅助类继承SQLiteOpenHelper类.

onCreate(SQLiteDatabase db): 当数据库被创建的时候，能够生成表，并创建视图跟触发器。
onUpgrade(SQLiteDatabse db, int oldVersion, int newVersion): 更新的时候可以删除表和创建新的表。


