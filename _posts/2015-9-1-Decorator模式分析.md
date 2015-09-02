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

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img91.png)

如上图所示，主要包括四个部分：

1. Component抽象组件，是一个接口或者是抽象类，就是定义我们最核心的对象，也就是最原始的对象。（注：在装饰模式中，必然有一个最基本、最核心、最原始的接口或者抽象类充当Component抽象组件）

2. ConcreteComponent具体组件，是最核心、最原始、最基本的接口或抽象类的实现，我们需要装饰的就是它。

3. Decorator装饰角色，一般是一个抽象类，实现接口或者抽象方法，它的属性里必然有一个private变量指向Component抽象组件。

4. 具体装饰角色，如上图中的ConcreteDecoratorA和ConcreteDecoratorB，我们要把我们最核心的、最原始的、最基本的东西装饰成其它东西。


## 装饰模式的优势

装饰类和被装饰类可以独立发展，而不会相互耦合。换句话说，Component类无须知道Decorator类，Decorator类是从外部来扩展Component类的功能，
而Decorator也不用知道具体的组件。
装饰模式是继承关系的一个替代方案。我们看装饰类Decorator，不管装饰多少层，返回的对象还是Component,装饰模式可以动态地扩展一个实现类的功能。



