---
layout: post
author: mxn
titile: java类型信息RTTI
category: 技术博文
tag: android
---

RTTI Run-Time Type Infomation（运行时类型信息），在Java运行时，RTTI维护类的相关信息，识别类和对象的信息。
多态(polymorphism)是基于RTTI实现的。RTTI的功能主要是由Class类实现的。

严格的说，反射也是一种形式的RTTI，不过，一般的文档资料中把RTTI和反射分开，因为一般的，大家认为RTTI指的是传统的RTTI
，通过继承和多态来实现，在运行时通过调用超类的方法来实现具体的功能（超类会自动实例化为子类，或使用instance of）。

传统的RTTI有3种实现方式：

1.向上转型或向下转型（upcasting and downcasting），在java中，向下转型（父类转成子类）需要强制类型转换。

2.Class对象（用了Class对象，不代表就是反射，如果只是用Class对象cast成指定的类，那就还是传统的RTTI）。

3.instanceof或isInstance()。

<!-- more -->

其中cast的用法如下所示：

{% highlight java %}
public class Test1 {
    public static void main(String[] args) throws ClassNotFoundException {
        Father f = new Child();
        Class<Child> cc = Child.class ;
        Child c = cc.cast(f) ; 
    }
}
class Father {
}
class Child extends Father {
}
    {% endhighlight %} 

这里 Child c = cc.cast(f) ; 等价于 Child c = (Child)f ;

传统的RTTI与反射最主要的区别：RTTI，编译器在编译时打开和检查.class文件。
而反射不需要，反射在运行时打开和检查.class文件。传统的RTTI使用转型或Instance形式实现，
但都需要指定要转型的类型，比如：

{% highlight java %}
public void rtti(Object obj){
    Toy toy = Toy(obj);
    // Toy toy = Class.forName("com.rtti.Toy")
    // obj instanceof Toy
}
{% endhighlight %} 

注意其中的obj虽然是被转型了，但在编译期，就需要知道要转成的类型Toy，也就是需要Toy的.class文件。
相对的，反射完全在运行时在通过Class类来确定类型，不需要提前加载Toy的.class文件。

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

使用forName(String str)有一个副作用：如果类没有被加载，调用它会触发类的static子句（静态初始化块）。与之相比，更好用的是类字面常量，
例如Child.class。支持编译时检查，所以不会抛出异常。使用类字面常量创建Class对象的引用与forName(String str)
不同，不会触发类的static子句（静态初始化块）。所以，更简单更安全更高效。类字面常量支持类、接口、数组、基本数据类型。


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

更详细的介绍可以参考：

[Java 类加载与初始化](http://www.cnblogs.com/zhguang/archive/2013/06/25/3154584.html)

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

若从表面看，Class的newInstance()方法似乎是克隆（clone()）一个对象的另一种手段。但两者是有区别的。利用newInstance()，
我们可在没有现成对象供“克隆”的情况下新建一个对象。就像上面的程序演示的那样，当时没有Toy对象，只有cy——即Toy的Class对象。
利用它可以实现“虚拟构建器”。在上述例子中，cy只是一个Class对象，编译期间并不知道进一步的类型信息。一旦新建了一个实例后，
可以得到Object对象。但那是一个Toy对象。

用newInstance()创建的类必须有一个无参数的构造方法。没有办法用newInstance()创建拥有非默认构建方法的对象，如果我们注释掉Toy() {}构造方法，
将会报错：java.lang.InstantiationException。


参考阅读：

[Thinking in Java中对RTTI的介绍](http://nicolas.blancpain.free.fr/Documents/Java/online/Chapter12.html)