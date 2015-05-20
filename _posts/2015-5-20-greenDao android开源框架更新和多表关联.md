---
layout: post
author: mxn
title: greenDao android开源框架更新和多表关联
category: 技术博文
tag: [android]
---

####更新操作

使用greenDao当android应用升级数据库新增表或者修改表，如何只是修改版本号数据会被清空。
需要自己修改SQLiteOpenHelper
1.找到greenDao生成的DaoMaster.java文件，里面有SQLiteOpenHelper实现
2.修改DevOpenHelper类里的   public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) 方法
{% highlight java %}
 @Override
        public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
            Log.i("greenDAO", "Upgrading schema from version " + oldVersion + " to " + newVersion + " by dropping all tables");
            dropAllTables(db, true);
            onCreate(db);
        }
     {% highlight  %}   
        
通过oldVersion newVersion 来判断需要创建表或alter表


####多表关联操作

http://greendao-orm.com/documentation/relations/

