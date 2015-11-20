---
layout: post
author: mxn
title: greenDao数据库更新和多表关联
category: 技术博文
tag: [android]
---

### 更新操作


使用greenDao当android应用升级数据库新增表或者修改表，如何只是修改版本号数据会被清空。
需要自己修改SQLiteOpenHelper：

1.找到greenDao生成的DaoMaster.java文件，里面有SQLiteOpenHelper实现

2.修改DevOpenHelper类里的   public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) 方法

{% highlight java %}
        @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            Log.i("greenDAO", "Upgrading schema from version " + oldVersion + " to " + newVersion + " by dropping all tables");
            dropAllTables(db, true);
            onCreate(db);
        }
     {% endhighlight  %}   
        
通过oldVersion newVersion 来判断需要创建表或alter表
http://blog.csdn.net/h3c4lenovo/article/details/43566169

### 多表关联操作

数据库的表可能涉及到一对一和一对多关系，在greenDAO中涉及到to-one和to-many关系。例如，你想在greenDAO中建立一个对多关系模型，你就
需要使用to-one和to-many关系。但是，请注意，to-one和to-many关系是不相关的，所以你必须同时更新。

#### 关系名称和多种关系




