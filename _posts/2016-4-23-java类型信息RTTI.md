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
        Child child = new Child();
        Class c2  = child.getClass();
        System.out.println(c2.getName());
    }
}
class Father {
}
class Child extends Father {
}
    {% endhighlight %} 
    
