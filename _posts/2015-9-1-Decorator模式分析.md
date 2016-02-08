---
layout: post
author: mxn
titile: Decorator模式分析
category: 技术博文
tag: android
---

## 装饰模式定义

装饰模式又名包装(Wrapper)模式，装饰模式以对客户透明的方式动态地给一个对象附加上更多的责任。换言之，
客户端并不会觉得对象在装饰前和装饰后有什么不同。装饰模式可以在不使用创造更多子类的情况下，将对象的功能加以扩展。

我们先来看下该设计模式的UML结构图

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img92.png)

如上图所示，主要包括四个部分：

1. Component抽象组件，是一个接口或者是抽象类，就是定义我们最核心的对象，也就是最原始的对象。（注：在装饰模式中，必然有一个最基本、最核心、最原始的接口或者抽象类充当Component抽象组件）

2. ConcreteComponent具体组件，是最核心、最原始、最基本的接口或抽象类的实现，我们需要装饰的就是它。

3. Decorator装饰角色，一般是一个抽象类，实现接口或者抽象方法，它的属性里必然有一个private变量指向Component抽象组件。

4. 具体装饰角色，如上图中的ConcreteDecoratorA和ConcreteDecoratorB，我们要把我们最核心的、最原始的、最基本的东西装饰成其它东西。

<!-- more -->

## 实现方式

假设情景：某人装扮自己形象，穿衣服，裤子，鞋子，戴帽子等来把自己给包装起来，需要把所需的功能按正确的顺序串联起来进行控制，我们应该如何设计才能做到呢？如下，先看下代码结构图：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img94.png)

1.先创建一个接口类：Component.java

    {% highlight java  %}
public interface Component {
    void show();
}
     {% endhighlight %}


2.创建一个具体的 ConcreteComponent 来实现 Component 接口：Person.java

    {% highlight java  %}
public class Person implements Component{
    private String name;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Person(String name){
        this.name = name;
    }
    @Override
    public void show() {
        System.out.println("装扮的" + name);
    }
}
     {% endhighlight %}


3.创建装饰类 Decorator 实现 Component 接口

    {% highlight java  %}
public class Decorator implements Component{
    private Component mComponent;
    public void decoratorObj(Component component){
        mComponent = component;
    }
    @Override
    public void show() {
        if(mComponent != null){
            mComponent.show();
        }
    }
}
     {% endhighlight %}

4.分别创建具体的装饰类：Jeans.java , Pelisse.java, Sandal.java ...等等，分别继承 Decorator.java 类：

    {% highlight java  %}
/** 牛仔裤 */
public class Jeans extends Decorator {
    @Override
    public void show(){
        System.out.println("穿牛仔裤");
        super.show();
    }
}
     {% endhighlight %}

其余类类似，在这里就省略了。

5.客户端测试类：

    {% highlight java  %}
/**
 * 装饰模式测试客户端
 * @author Andy.Chen
 *
 */
public class DecoratorClient {
    public static void main(String[] args) {
        System.out.println("Welcome to Andy.Chen Blog!" +"\n"
                   +"Decorator Patterns." +"\n");
        Person mPerson = new Person("Andy");
        Sandal mSandal = new Sandal();
        Jeans mJeans = new Jeans();
        TShirt mShirt = new TShirt();
        mShirt.decoratorObj(mPerson);
        mJeans.decoratorObj(mShirt);
        mSandal.decoratorObj(mJeans);
        mSandal.show();
    }
}
     {% endhighlight %}

测试显示输出的结果如下：

    {% highlight c  %}
        穿凉鞋
        穿牛仔裤
        穿T-Shirt
        装扮的Andy
     {% endhighlight %}

## 装饰模式的优势

装饰类和被装饰类可以独立发展，而不会相互耦合。换句话说，Component类无须知道Decorator类，Decorator类是从外部来扩展Component类的功能，
而Decorator也不用知道具体的组件。
装饰模式是继承关系的一个替代方案。我们看装饰类Decorator，不管装饰多少层，返回的对象还是Component,装饰模式可以动态地扩展一个实现类的功能。

## 装饰模式在Android源码中的应用

在Android源码中，其中一个比较经典的使用到装饰模式的就是由Context抽象类扩展出的ContextWrapper的设计。继承结构如下图所示：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img93.png)

1. Context就是我们的抽象组件，它提供了应用运行的基本环境，是各组件和系统服务通信的桥梁，隐藏了应用与系统服务通信的细节，简化了上层应用的开发。所以Contex就是“装饰模式”里的Component。

2. Context类是个抽象类，android.app.ContextImpl派生实现了它的抽象接口。ContextImpl对象会与Android框架层的各个服务（包括组件管理服务、资源管理服务、安装管理服务等）建立远程连接，通过对Android进程间的通信机制（IPC）和这些服务进行通信。所以ContextImpl就是“装饰模式”里的ConcreteComponent。

3. 如果上层应用期望改变Context接口的实现，就需要使用android.content.ContextWrapper类，它派生自Context，其中具体实现都是通过组合的方式调用ContextImpl类的实例（在ContextWrapper中的private属性mBase）来完成的。这样的设计，使得ContextImpl与ContextWrapper子类的实现可以单独变化，彼此独立。所以可以看出ContextWrapper就是“装饰模式”里的Decorator。

4. Android的界面组件Activity、服务组件Service以及应用基类Application都派生于ContextWrapper，它们可以通过重载来修改Context接口的实现。所以可以看出Activity、服务组件Service以及应用基类Application就是“装饰模式”里的具体装饰角色A、B、C。

注：上图可以看出界面组件基类android.app.Activity添加了界面绘制相关的实现，增加了处理界面事件的相关接口。它存放界面中各控件的对象，并与窗口管理服务建立连接，传递界面相关的事件和操作。

## 装饰模式和代理模式的区别

装饰模式和静态代理极为相似。

相同点：都是为被代理(被装饰)的类扩充新的功能。
不同点：代理模式具有控制被代理类的访问等性质，而装饰模式紧紧是单纯的扩充被装饰的类。所以区别仅仅在是否对被代理/被装饰的类进行了控制而已。

## 适用性

在初次学习Decorator模式的时候，觉得使用Deccorator或许并不好用，Decorator让对象被装饰进行包装，得到的将是一个被装饰后的对象，而已不是对象本身。
虽然可以通过Decorator传递请求，但势必让事情变得更麻烦。故通常我们仍然需要操作的是原对象本身。这样一来需要维护两个对象。
结果的确是这样：需要通过Decorator传递对原对象的请求，让结构层次变得较深稍微复杂，如果使用继承（假设让某Commponent继承某Decorator），构造一个更复杂的类，我们就可以直接维护一个对象即可。
但是使用继承我们实际上需要维护的职责功能没有变，仍然是两个部分：Decorator和Component；在大多数情况下我们可能根本不需要Decorator而只需要Commponent部分，没能做到在需要时动态添加功能；
使用Decorator去动态的包装Component，可以做到“即付即用”的方法添加职责，也可以逐步的个一个简单对象添加职责构造复杂的对象；对象的维护具有层次性：客户端仅仅需要维护Decorator即可，
被装饰的对象交给Decorator来维护即可，还降低了复杂性。
做到如此最重要的是：Decorator是一个透明的包装；也就说被包装后的对象和被包装的对象具有一致性，在使用装饰对象时就像使用原对象一样，而不需要加以区分；故要使Decorator和Component具有一致的接口，
从同一个父类派生下来——保持接口的一致性。Decorator是对象的装饰包装，Decorator如果太复杂，有可能破坏Decorator与Component的一致性，
故Decorator尽量保持简单所添加的职责单一。所以Decorator要做到：动态的包装对象，添加对象新的职责，保持与原对象一致性。

### 其他参考

Proxy, Decorator, Adapter, and Bridge are all variations on "wrapping" a class. But their uses are different.

1.Proxy could be used when you want to lazy-instantiate an object, or hide the fact that you're
calling a remote service, or control access to the object.

2.Decorator is also called "Smart Proxy." This is used when you want to add functionality to an
object, but not by extending that object's type. This allows you to do so at runtime.

3.Adapter is used when you have an abstract interface, and you want to map that interface to
another object which has similar functional role, but a different interface.

4.Bridge is very similar to Adapter, but we call it Bridge when you define both the abstract
interface and the underlying implementation. I.e. you're not adapting to some legacy or third-party code, you're the designer of all the code but you need to be able to swap out different implementations.

5.Facade is a higher-level (read: simpler) interface to a subsystem of one or more classes.
Suppose you have a complex concept that requires multiple objects to represent. Making changes to that set of objects is confusing, because you don't always know which object has the method you need to call. That's the time to write a Facade that provides high-level methods for all the complex operations you can do to the collection of objects. Example: a Domain Model for a school section, with methods like countStudents(), reportAttendance(), assignSubstituteTeacher(), and so on.
