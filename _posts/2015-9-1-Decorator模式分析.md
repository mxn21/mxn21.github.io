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

