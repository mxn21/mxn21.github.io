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

Java程序在运行之前并没有被完全加载，各个部分是在需要时才被加载的。

为了使用类而作的准备包含三步：

1.加载。由classloader查找class字节码文件，创建一个Class对象。
2.链接：验证字节码文件，为静态域分配存储空间，如果必需的话，会解析这个类创建的对其他类的所有引用（比如说该类持有static域）。
3.初始化：初始化父类，执行静态初始化器和静态初始化块。

其中静态初始化器可以理解为静态域在定义处的初始化，如：static Dog d = new Dog(0);。

final static成员和static成员的在以上三个过程中不一样，final static成员被称为“编译器常量”，在编译时已经被赋值，
所以可以在类加载前就进行访问，而静态成员（非final）需要在类加载后、class对象初始化之后赋值。

当Java创建某个类的对象，jvm虚拟机的classLoader会检测对象对应的Class对象是否已加载,
比如Child类对象时，Java会检查内存中是否有相应的Class对象。如果内存中没有相应的Class对象，会依据相关途径查询对应.class文件
（如 通过classPath在本地文件系统进行查找,在获取到.class文件之后会对文件进行有效验证，之后会依据Class对象进行详细类型对象的创建。
在Class对象加载成功后，其他Child对象的创建和相关操作都将参照该Class对象。


### 举例说明

   {% highlight java %}
interface HasBatteries {}
interface Waterproof {}
interface Shoots {}
class Toy {
    Toy() {}
    Toy(int i) {}
}
class FancyToy extends Toy implements HasBatteries, Waterproof, Shoots {
    FancyToy() { super(1); }
}
public class ToyTest {
    static void printInfo(Class cc) {
        System.out.println("Class name: " + cc.getName() +
                " is interface? [" + cc.isInterface() + "]");
    }
    public static void main(String[] args) {
        Class c = null;
        try {
            c = Class.forName("com.souly.myapplication.test.FancyToy");
        } catch(ClassNotFoundException e) {
            System.out.println("Can't find FancyToy");
            System.exit(1);
        }
        printInfo(c);
        Class[] faces = c.getInterfaces();
        for(int i = 0; i < faces.length; i++)
            printInfo(faces[i]);
        Class cy = c.getSuperclass();
        Object o = null;
        try {
            // Requires default constructor:
            o = cy.newInstance(); // (*1*)
        } catch(InstantiationException e) {
            System.out.println("Cannot instantiate");
            System.exit(1);
        } catch(IllegalAccessException e) {
            System.out.println("Cannot access");
            System.exit(1);
        }
        printInfo(o.getClass());
    }
}
    {% endhighlight %}
     
运行后输出结果如下：

   {% highlight c %}
   Class name: com.souly.myapplication.test.FancyToy is interface? [false]
   Class name: com.souly.myapplication.test.HasBatteries is interface? [true]
   Class name: com.souly.myapplication.test.Waterproof is interface? [true]
   Class name: com.souly.myapplication.test.Shoots is interface? [true]
   Class name: com.souly.myapplication.test.Toy is interface? [false]
    {% endhighlight %}

从中可以看出，class FancyToy相当复杂，因为它从Toy中继承，并实现了HasBatteries，Waterproof以及ShootsThings的接口。在main()方法中
有一个Class对象，它通过Class.forName()初始化成FancyToy Class。Class.getInterfaces方法会返回Class对象的一个数组，用于表示Class对象内的接口。
若有一个Class对象，也可以用getSuperclass()查询该对象的直接父类是什么。当然，这种做会返回一个Class对象，可用它作进一步的查询。
这意味着在运行期的时候，完全有机会调查到对象的完整层次结构。

若从表面看，Class的newInstance()方法似乎是克隆（clone()）一个对象的另一种手段。但两者是有区别的。利用newInstance()，我们可在没有现成对象供“克隆”的情况下新建一个对象。
就像上面的程序演示的那样，当时没有Toy对象，只有cy——即Toy的Class对象。利用它可以实现“虚拟构建器”。换言之，我们表达：“尽管我不知道你的准确类型是什么，但请你无论如何都正确地创建自己。”