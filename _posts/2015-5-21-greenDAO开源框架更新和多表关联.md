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
     
#### To-Many关系模型

To-many关系和To-One关系类似，除了外键是放置在目标表中。让我们看看客户/订单的例子，客户可以有多个订单，所以我们用To-Many关系模型，
在数据库中，在订单表中创建customer ID列，来创建1:N关系。这样的话，就可以使用客户的id查询客户的所有的订单。

在greenDAO中建立to-many模型的方法和数据库中的操作类似，首先需要在目标实体中增加一个属性，用于关联To-many关系中的资源实体，然后使用
这个属性，添加到资源实体的To-many关系。

假设我们有一个客户和一个订单实体，我们想把订单连接到一个客户。以下代码将添加到客户实体的To-Many关系：

    {% highlight java %}
Property customerId = order.addLongProperty("customerId").notNull().getProperty();
ToMany customerToOrders = customer.addToMany(order, customerId);
customerToOrders.setName("orders"); // Optional
customerToOrders.orderAsc(orderDate); // Optional
     {% endhighlight  %} 
     
这样，我们可以在客户类中简单的调用生成的getOrders()方法获取订单。

#### To-Many关系的解析和更新

To-Many关系在第一次请求时懒汉式解析，之后，关联的实体被缓存在资源实体的List中。以后再调用get方法不查询数据库。

注意更新To-Many关系模型需要一些额外的工作。因为To-Many的列表已经被缓存，当将关联实体添加到数据库中时，它们不被更新。
下面的代码说明了这种行为：

    {% highlight java %}
List orders1 = customer.getOrders();
int size1 = orders1.size();

Order order = new Order();
order.setCustomerId(customer.getId());
daoSession.insert(order);

Listorders2 = customer.getOrders();
// size1 == orders2.size(); // NOT updated
// orders1 == orders2; // SAME list object
     {% endhighlight  %} 

因为缓存，您应该手动添加新的关系实体到To-Many关系资源实体的list。这就涉及到如何插入新的关系实体。

1.取得to-many list集合(必须在新实体建立前执行，因为我们不知道我们是否从最新的结果中取得缓存，这样执行后我们就知道了已经缓存了)
2.创造一个实体对象(作为many)
3.在新的实体中设置外键
4.使用insert插入新对象
5.添加新的对象到list

代码如下：

    {% highlight java %}
List orders = customer.getOrders();
newOrder.setCustomerId(customer.getId());
daoSession.insert(newOrder);
orders.add(newOrder);
     {% endhighlight  %} 
     
注意，getOrders在insert之前调用，确保list被缓存。如果getOrders在insert之后，如果万一list没有在之前被缓存，那么新的对象
会在list中出现两次。

同样，您可以删除关联实体：

    {% highlight java %}
List orders = customer.getOrders();
daoSession.delete(newOrder);
orders.remove(newOrder);
     {% endhighlight  %} 
     
有时，在关联实体添加或删除后手动更新所有To-Many关系可能是繁琐的，甚至是不可能的。为了补救，greendao有reset方法来清除缓存列表。
如果to-many关系可能发生潜在变化，你可以重载关联实体的list：

    {% highlight java %}
customer.resetOrders();
List orders2 = customer.getOrders();
     {% endhighlight  %} 
     
     
#### 双向1:N关系

有时你想实现双向1:n关系，在greendao中，你要添加to-one和to-many关系来实现这个。下面的例子使用我们之前例子的客户和订单，
显示了完整的建模实体的过程。这次我们使用customerId属性创建两种关系。

 {% highlight java %}
Entity customer = schema.addEntity("Customer");
customer.addIdProperty();
customer.addStringProperty("name").notNull();

Entity order = schema.addEntity("Order");
order.setTableName("ORDERS"); // "ORDER" is a reserved keyword
order.addIdProperty();
Property orderDate = order.addDateProperty("date").getProperty();
Property customerId = order.addLongProperty("customerId").notNull().getProperty();
order.addToOne(customer, customerId);

ToMany customerToOrders = customer.addToMany(order, customerId);
customerToOrders.setName("orders");
customerToOrders.orderAsc(orderDate);
{% endhighlight  %} 

让我们假设有一个订单的实体。利用这两个关系，我们可以得到客户的信息和客户的所有订单。

    {% highlight java %}
List allOrdersOfCustomer = order.getCustomer().getOrders();
     {% endhighlight  %} 
     
     
#### Many-to-Many关系(n:m)

在数据库关系模型中M，N是利用连接表。该表的实体具有一系列外键，这些外键关联到每一个关联表。目前greendao不直接支持Many-to-Many关系，
你可以用单独的实体来建模连接表。在实践中，你通常有一个具有额外属性的关系实体，你可能想这样做。在未来的版本，
greendao可能会支持N：M关系。


#### 树形关系

你可以通过建立一个具有指向自己的to-one和to-many关系的实体来建立一个树形关系模型，