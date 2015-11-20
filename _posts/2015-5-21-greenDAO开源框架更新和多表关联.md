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

每一个关系都有一个名字，名字用在维持关系的实体类的相应的属性上，默认名称是目标实体的名称。这个名字可以用setName方法重写。
请记住，如果一个实体与同一个目标实体有多个关系，默认名称不能唯一。

#### To-One关系模型

在greenDAO generator中建模时，必须使一个属性作为外键，使用这个属性，你可以用Entity.addToOne方法增加to-one关系。
addToOne方法的参数是另一个实体，和本实体的外键属性。

    {% highlight java %}
/**
     * Adds a to-one relationship to the given target entity using the given given foreign key property (which belongs
     * to this entity).
     */
    public ToOne addToOne(Entity target, Property fkProperty) {
        if(protobuf) {
            throw new IllegalStateException("Protobuf entities do not support realtions, currently");
        }

        Property[] fkProperties = { fkProperty };
        ToOne toOne = new ToOne(schema, this, target, fkProperties, true);
        toOneRelations.add(toOne);
        return toOne;
    }

    /** Convenience for {@link #addToOne(Entity, Property)} with a subsequent call to {@link ToOne#setName(String)}. */
    public ToOne addToOne(Entity target, Property fkProperty, String name) {
        ToOne toOne = addToOne(target, fkProperty);
        toOne.setName(name);
        return toOne;
    }
     {% endhighlight  %}   
     
     
例如：user有一个picture属性，user和picture都是普通实体

    {% highlight java %}
// The variables "user" and "picture" are just regular entities
Property pictureIdProperty = user.addLongProperty("pictureId").getProperty();
user.addToOne(picture, pictureIdProperty);
     {% endhighlight  %}   

这样就使user有一个Picture属性，并且可以直接操作Picture对象，user类具有Picture属性的getPicture/setPicture方法。
to-one关系中的getter方法在第一次加载目标实体的时候是懒汉式加载，之后的访问将返回先前已解析的对象。

注意外键属性（“pictureid”）和实体对象的属性（“picture”）绑在一起。如果你改变了pictureid，下一次调用getPicture()的时候就
会用更新之后的id重新解析Picture实体。同样，如果设置了一个新的picture实体，pictureId属性也会被更新。

greenDAO同样支持在to-one中的饿汉式加载，这会在一次数据库查询中解析一个类的所有to-one关系，这会在你关联实体类时有不错的表现，
目前你可以使用loadDeep和queryDeep方法，以后可能会改变。

#### 关系名称和多种关系

每一关系都有一个名字，它被用于实体的相应的属性中。默认名称是目标实体的名称。这个名字可以用setName方法重写。
如果一个实体与同一个目标实体有多个关系，默认名称不是唯一的。在这种情况下，必须显式地指定关系名。

让我们扩展前面的例子，让user还拥有一个缩略图(thumbnail picture)。因为图片和缩略图都是指同一个实体(picture)，所以会有一个名称冲突。
因此，我们将第二个关系重新命名为“缩略图”：

    {% highlight java %}
Property pictureIdProperty = user.addLongProperty("pictureId").getProperty();
Property thumbnailIdProperty = user.addLongProperty("thumbnailId").getProperty();
user.addToOne(picture, pictureIdProperty);
user.addToOne(picture, thumbnailIdProperty, "thumbnail");
     {% endhighlight  %} 
     
