---
layout: post
author: mxn
titile: java类型信息RTTI
category: 技术博文
tag: android
---

RTTI Run-Time Type Infomation（运行时类型信息），在Java运行时，RTTI维护类的相关信息，识别类和对象的信息。
多态(polymorphism)是基于RTTI实现的。RTTI的功能主要是由Class类实现的。


### Class类

Class类是"类的类"(class of classes)。如果说类是对象的抽象和集合的话，那么Class类就是对类的抽象和集合。
每一个Class类的对象代表一个其他的类。比如下面的程序中，Class类的对象c1代表了Father类，c2代表了Child类。

    {% highlight java %}
public class Test1 {
    public static void main(String[] args)
    {
        Father father = new Father();
        Class c1  = father.getClass();
        System.out.println(c1.getName());
        Father child = new Child();
        Class c2  = child.getClass();
        System.out.println(c2.getName());
    }
}
class Father {
}
class Child extends Father {
}
    {% endhighlight %} 
    
当我们调用对象的getClass()方法时，就得到对应Class对象的引用。
在c2中，即使我们将child对象的引用向上转换为Father对象的引用，对象所指向的Class类对象依然是Child。

Java中每个对象都有相应的Class类对象，因此，我们随时能通过Class对象知道某个对象“真正”所属的类。无论我们对引用进行怎样的类型转换，
对象本身所对应的Class对象都是同一个。当我们通过某个引用调用方法时，Java总能找到正确的Class类中所定义的方法，并执行该Class类中的代码。
由于Class对象的存在，Java不会因为类型的向上转换而迷失。这就是多态的原理。

除了getClass()方法外，我们还有其他方式调用Class类的对象。

   {% highlight java %}
public class Test1 {
    public static void main(String[] args) throws ClassNotFoundException {
        Class c1  = Class.forName("com.souly.myapplication.test.Father");
        System.out.println(c1.getName());
        Class c2  = Child.class ;
        System.out.println(c2.getName());
    }
}
class Father {
}
class Child extends Father {
}
  {% endhighlight %} 
  
上面显示了两种方式:

1.forName()方法接收一个字符串作为参数，该字符串是类的名字。这将返回相应的Class类对象。
2.Child.class方法是直接调用类的class成员。这将返回相应的Class类对象。

### Class类的加载

当Java创建某个类的对象，比如Child类对象时，Java会检查内存中是否有相应的Class对象。如果内存中没有相应的Class对象，
那么Java会在.class文件中寻找Child类的定义，并加载Child类的Class对象。在Class对象加载成功后，
其他Child对象的创建和相关操作都将参照该Class对象。

