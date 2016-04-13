---
layout: post
author: mxn
titile: Java中的volatile关键字
category: 技术博文
tag: java
---

Java 语言中的 volatile 变量可以被看作是一种 “程度较轻的 synchronized”；与 synchronized 块相比，volatile 变量所需的编码较少，
并且运行时开销也较少，但是它所能实现的功能也仅是 synchronized 的一部分。

当volatile用于一个作用域时，Java保证如下：

1.（适用于Java所有版本）读和写一个volatile变量有全局的排序。也就是说每个线程访问一个volatile作用域时会在继续执行之前读取它的当前值，
而不是（可能）使用一个缓存的值。(但是并不保证经常读写volatile作用域时读和写的相对顺序，也就是说通常这并不是有用的线程构建)。

2.（适用于Java5及其之后的版本）volatile的读和写建立了一个happens-before关系，类似于申请和释放一个互斥锁。

使用volatile会比使用锁更快，但是在一些情况下它不能工作。volatile使用范围在Java5中得到了扩展，特别是双重检查锁定现在能够正确工作。
volatile可以用在任何变量前面，但不能用于final变量前面，因为final型的变量是禁止修改的。也不存在线程安全的问题。

### 原子操作

原子（atom）本意是“不能被进一步分割的最小粒子”，而原子操作（atomic operation）意为"不可被中断的一个或一系列操作" 。
原子操作在操作完毕之前不会线程调度器中断。在Java中，对除了long和double之外的基本类型的简单操作都具有原子性。
简单操作就是赋值或者return。比如”a = 1;“和 “return a;”这样的操作都具有原子性。但是在Java中，类似“a += b”这样的操作不具有原子性，
所以如果方法不是同步的就会出现难以预料的结果。在某些JVM中”a += b”可能要经过这样三个步骤：
1.取出a和b。2.计算a+b。3.将计算结果写入内存。

如果有两个线程t1,t2在进行这样的操作。t1在第二步做完之后还没来得及把数据写回内存就被线程调度器中断了，于是t2开始执行，
t2执行完毕后t1又把没有完成的第三步做完。这个时候就出现了错误，相当于t2的计算结果被无视掉了。
类似的，像“a++”这样的操作也都不具有原子性。所以在多线程的环境下一定要记得进行同步操作。

要搞清楚这个问题，首先应该明白计算机内部都做什么了。比如做了一个i++操作，计算机内部做了三次处理：读取－修改－写入。
同样，对于一个long型数据，做了个赋值操作，在32系统下需要经过两步才能完成，先修改低32位，然后修改高32位。
 
假想一下，当将以上的操作放到一个多线程环境下操作时候，有可能出现的问题，是这些步骤执行了一部分，而另外一个线程就已经引用了变量值，
这样就导致了读取脏数据的问题。


### happens-before关系

Java语言中有一个“先行发生”（happen—before）的规则，它是Java内存模型中定义的两项操作之间的偏序关系，如果操作A先行发生于操作B，
其意思就是说，在发生操作B之前，操作A产生的影响都能被操作B观察到，“影响”包括修改了内存中共享变量的值、发送了消息、调用了方法等，
它与时间上的先后发生基本没有太大关系。这个原则特别重要，它是判断数据是否存在竞争、线程是否安全的主要依据。

举例来说，假设存在如下三个线程，分别执行对应的操作:

线程A中执行如下操作：i=1。

线程B中执行如下操作：j=i。

线程C中执行如下操作：i=2。

假设线程A中的操作”i=1“ happen—before线程B中的操作“j=i”，那么就可以保证在线程B的操作执行后，变量j的值一定为1，
即线程B观察到了线程A中操作“i=1”所产生的影响；现在，我们依然保持线程A和线程B之间的happen—before关系，同时线程C出现在
了线程A和线程B的操作之间，但是C与B并没有happen—before关系，那么j的值就不确定了，线程C对变量i的影响可能会被线程B观察到，
也可能不会，这时线程B就存在读取到不是最新数据的风险，不具备线程安全性。

下面是Java内存模型中的八条可保证happen—before的规则，它们无需任何同步器协助就已经存在，可以在编码中直接使用。
如果两个操作之间的关系不在此列，并且无法从下列规则推导出来的话，它们就没有顺序性保障，虚拟机可以对它们进行随机地重排序。

1、程序次序规则：在一个单独的线程中，按照程序代码的执行流顺序，（时间上）先执行的操作happen—before（时间上）后执行的操作。

2、管理锁定规则：一个unlock操作happen—before后面（时间上的先后顺序，下同）对同一个锁的lock操作。

3、volatile变量规则：对一个volatile变量的写操作happen—before后面对该变量的读操作。

4、线程启动规则：Thread对象的start（）方法happen—before此线程的每一个动作。

5、线程终止规则：线程的所有操作都happen—before对此线程的终止检测，可以通过Thread.join（）方法结束、Thread.isAlive（）
的返回值等手段检测到线程已经终止执行。

6、线程中断规则：对线程interrupt（）方法的调用happen—before发生于被中断线程的代码检测到中断时事件的发生。

7、对象终结规则：一个对象的初始化完成（构造函数执行结束）happen—before它的finalize（）方法的开始。

8、传递性：如果操作A happen—before操作B，操作B happen—before操作C，那么可以得出A happen—before操作C。

#### 时间上先后顺序和happen—before原则

“时间上执行的先后顺序”与“happen—before”之间有何不同呢？
1、首先来看操作A在时间上先与操作B发生，是否意味着操作A happen—before操作B？

一个常用来分析的例子如下：

    {% highlight java %} 
	private int value = 0;
	public int get(){
		return value;
	}
	public void set(int value){
		this.value = value;
	}
}
   {% endhighlight %}
   
假设存在线程A和线程B，线程A先（时间上的先）调用了setValue（3）操作，然后（时间上的后）线程B调用了同一对象的getValue（）方法，
那么线程B得到的返回值一定是3吗？
对照以上八条happen—before规则，发现没有一条规则适合于这里的value变量，从而我们可以判定线程A中的setValue（3）操作与
线程B中的getValue（）操作不存在happen—before关系。因此，尽管线程A的setValue（3）在操作时间上先于操作B的getvalue（），
但无法保证线程B的getValue（）操作一定观察到了线程A的setValue（3）操作所产生的结果，也即是getValue（）的返回值不一定为3
（有可能是之前setValue所设置的值）。这里的操作不是线程安全的。

因此，“一个操作时间上先发生于另一个操作”并不代表“一个操作happen—before另一个操作”。

解决方法：可以将setValue（int）方法和getValue（）方法均定义为synchronized方法，也可以把value定义为volatile变量
（value的修改并不依赖value的原值，符合volatile的使用场景），分别对应happen—before规则的第2和第3条。注意，
只将setValue（int）方法和getvalue（）方法中的一个定义为synchronized方法是不行的，必须对同一个变量的所有读写同步，
才能保证不读取到陈旧的数据，仅仅同步读或写是不够的 。

2、其次来看，操作A happen—before操作B，是否意味着操作A在时间上先与操作B发生？

如果A happens- before B，JMM并不要求A一定要在B之前执行。JMM仅仅要求前一个操作（执行的结果）对后一个操作可见，
且前一个操作按顺序排在第二个操作之前。CPU是可以不按我们写代码的顺序执行内存的存取过程的，也就是指令会乱序或并行运行，
只有上面的happens-before所规定的情况下，才保证顺序性，确保任何内存的写，对其他语句都是可见的。

假设同一个线程执行上面两个操作：操作A：x=1和操作B：y=2。根据happen—before规则的第1条，操作A happen—before 操作B，
但是由于编译器的指令重排序（Java语言规范规定了JVM线程内部维持顺序化语义，也就是说只要程序的最终结果等同于它在严格的顺序化环境下的结果，
那么指令的执行顺序就可能与代码的顺序不一致。这个过程通过叫做指令的重排序。指令重排序存在的意义在于：JVM能够根据处理器的特性
（CPU的多级缓存系统、多核处理器等）适当的重新排序机器指令，使机器指令更符合CPU的执行特点，最大限度的发挥机器的性能。
在没有同步的情况下，编译器、处理器以及运行时等都可能对操作的执行顺序进行一些意想不到的调整）等原因，操作A在时间上有可能后于操作B被
处理器执行，但这并不影响happen—before原则的正确性。因此，“一个操作happen—before另一个操作”并不代表“一个操作时间上先发生于另一个操作”。

最后，一个操作和另一个操作必定存在某个顺序，要么一个操作或者是先于或者是后于另一个操作，或者与两个操作同时发生。
同时发生是完全可能存在的，特别是在多CPU的情况下。而两个操作之间却可能没有happen-before关系，也就是说有可能发生这样的情况，
操作A不happen-before操作B，操作B也不happen-before操作A，用数学上的术语happen-before关系是个偏序关系。
两个存在happen-before关系的操作不可能同时发生，一个操作A happen-before操作B，它们必定在时间上是完全错开的，
这实际上也是同步的语义之一（独占访问）。

举例说明如下：

    {% highlight java %} 
public class Test {
	private int a = 0;
	private long b = 0;
	public void set() {
		a = 1;
		b = -1;
	}
	public void check() {
		if (! ((b == 0) || (b == -1 && a == 1))
			throw new Exception("check Error!");
	}
}
   {% endhighlight %}
   
对于set()方法的执行： 
1. 编译器可以重新安排语句的执行顺序，这样b就可以在a之前赋值。如果方法是内嵌的(inline)，编译器还可以把其它语句重新排序。 
2. 处理器可以改变这些语句的机器指令的执行顺序，甚到同时执行这些语句。 
3. 存储系统(由于被缓存控制单元控制)也可以重新安排对应存储单元的写操作顺序，这些写操作可能与其他计算和存储操作同时发生。 
4. 编译器，处理器和存储系统都可以把这两条语句的机器指令交叉执行。 
例如：在一台32位的机器上，可以先写b的高位，然后写a，最后写b的低位，(注：b为long类型，在32位的机器上分高低位存储) 
5. 编译器，处理器和存储系统都可以使对应于变量的存储单元一直保留着原来的值， 以某种方式维护相应的值(例如，在CPU的寄存器中)
以保证代码正常运行，直到下一个check调用才更新。 

在单线程(或同步)的情况下，上面的check()永远不会报错， 但非同步多线程运行时却很有可能。
并且，多个CPU之间的缓存也不保证实时同步，也就是说你刚给一个变量赋值，另一个线程立即获取它的值，可能拿到的却是旧值(或null)， 
因为两个线程在不同的CPU执行，它们看到的缓存值不一样，只有在synchronized或volatile或final的性况下才能保证正确性。

例如：

    {% highlight java %} 
    public class Test {
        private int n;
        public void set(int n) {
            this.n = n;
        }
        public void check() {
            if (n != n)
                throw new Exception("check Error!");
        }
    }
    {% endhighlight %}
    
check()中的 n != n 好像永远不会成立，因为他们指向同一个值，但非同步时却很有可能发生。

### volatile使用情形

只能在有限的一些情形下使用 volatile 变量替代锁。要使 volatile 变量提供理想的线程安全，必须同时满足下面两个条件：

1.对变量的写操作不依赖于当前值。

2.该变量没有包含在具有其他变量的不变式中。

#### 模式1：状态标志

也许实现 volatile 变量的规范使用仅仅是使用一个布尔状态标志，用于指示发生了一个重要的一次性事件，例如完成初始化或请求停机。
很多应用程序包含了一种控制结构，形式为 “在还没有准备好停止程序时再执行一些工作”，如下所示：

    {% highlight java %} 
volatile boolean shutdownRequested;
...
public void shutdown() { shutdownRequested = true; }
public void doWork() { 
    while (!shutdownRequested) { 
        // do stuff
    }
}
    {% endhighlight %}
    
很可能会从循环外部调用 shutdown() 方法，即在另一个线程中。因此，需要执行某种同步来确保正确实现 shutdownRequested 变量的可见性。
然而，使用 synchronized 块编写循环要比使用如上所示的 volatile 状态标志编写麻烦很多。由于 volatile 
简化了编码，并且状态标志并不依赖于程序内任何其他状态，因此此处非常适合使用 volatile。

#### 模式2：一次性安全发布（one-time safe publication）

缺乏同步会导致无法实现可见性，这使得确定何时写入对象引用而不是原语值变得更加困难。在缺乏同步的情况下，可能会遇到某个对象引用的更新值
（由另一个线程写入）和该对象状态的旧值同时存在。（这就是造成著名的双重检查锁定（double-checked-locking）问题的根源，
其中对象引用在没有同步的情况下进行读操作，产生的问题是您可能会看到一个更新的引用，但是仍然会通过该引用看到不完全构造的对象）。
实现安全发布对象的一种技术就是将对象引用定义为 volatile 类型。下面的代码展示了一个示例，其中后台线程在启动阶段从数据库加载一些数据。
其他代码在能够利用这些数据时，在使用之前将检查这些数据是否曾经发布过。

    {% highlight java %} 
public class BackgroundFloobleLoader {
    public volatile Flooble theFlooble;
    public void initInBackground() {
        // do lots of stuff
        theFlooble = new Flooble();  // this is the only write to theFlooble
    }
public class SomeOtherClass {
    public void doWork() {
        while (true) { 
            // do some stuff...
            // use the Flooble, but only if it is ready
            if (floobleLoader.theFlooble != null) 
                doSomething(floobleLoader.theFlooble);
        }
    }
}
    {% endhighlight %}
    
如果 theFlooble 引用不是 volatile 类型，doWork() 中的代码在解除对 theFlooble 的引用时，将会得到一个不完全构造的 Flooble。
该模式的一个必要条件是：被发布的对象必须是线程安全的，或者是有效的不可变对象（有效不可变意味着对象的状态在发布之后永远不会被修改）。
volatile 类型的引用可以确保对象的发布形式的可见性，但是如果对象的状态在发布后将发生更改，那么就需要额外的同步。

#### 模式3：独立观察（independent observation）

安全使用 volatile 的另一种简单模式是：定期 “发布” 观察结果供程序内部使用。例如，假设有一种环境传感器能够感觉环境温度。
一个后台线程可能会每隔几秒读取一次该传感器，并更新包含当前文档的 volatile 变量。然后，其他线程可以读取这个变量，
从而随时能够看到最新的温度值。
使用该模式的另一种应用程序就是收集程序的统计信息。下面展示了身份验证机制如何记忆最近一次登录的用户的名字。将反复使用 
lastUser 引用来发布值，以供程序的其他部分使用。

    {% highlight java %} 
    public class UserManager {
        public volatile String lastUser;
        public boolean authenticate(String user, String password) {
            boolean valid = passwordIsValid(user, password);
            if (valid) {
                User u = new User();
                activeUsers.add(u);
                lastUser = user;
            }
            return valid;
        }
    }
    {% endhighlight %}
    
该模式是前面模式的扩展；将某个值发布以在程序内的其他地方使用，但是与一次性事件的发布不同，
这是一系列独立事件。这个模式要求被发布的值是有效不可变的 —— 即值的状态在发布后不会更改。使用该值的代码需要清楚该值可能随时发生变化。

