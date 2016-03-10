---
layout: post
author: mxn
titile: 理解ThreadLocal
category: 技术博文
tag: android
---

ThreadLocal类用来提供线程内部的局部变量。这种变量在多线程环境下访问(通过get或set方法访问)时能保证各个线程里的变量相对独立于其他线程内的变量。

ThreadLocal不能使用原子类型，只能使用Object类型。
threadLocal为每一个线程都提供了变量的副本， 使得每个线程在某一时间访问到的并不是同一个对象，
这样就隔离了多个线程对数据的数据共享。

ThreadLocal是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，数据存储以后，只有在指定线程中可以获取到存储的数据，
对于其它线程来说无法获取到数据。在日常开发中用到ThreadLocal的地方较少，但是在某些特殊的场景下，通过ThreadLocal可以轻松地实
现一些看起来很复杂的功能，这一点在Android的源码中也有所体现，比如Looper、ActivityThread以及AMS中都用到了ThreadLocal。
具体到ThreadLocal的使用场景，这个不好统一地来描述，一般来说，当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候，
就可以考虑采用ThreadLocal。比如对于Handler来说，它需要获取当前线程的Looper，很显然Looper的作用域就是线程并且不同线程具有
不同的Looper，这个时候通过ThreadLocal就可以轻松实现Looper在线程中的存取，如果不采用ThreadLocal，那么系统就必须提供一个
全局的哈希表供Handler查找指定线程的Looper，这样一来就必须提供一个类似于LooperManager的类了，但是系统并没有这么做而是选择
了ThreadLocal，这就是ThreadLocal的好处。

ThreadLocal另一个使用场景是复杂逻辑下的对象传递，比如监听器的传递，有些时候一个线程中的任务过于复杂，
这可能表现为函数调用栈比较深以及代码入口的多样性，在这种情况下，我们又需要监听器能够贯穿整个线程的执行过程，这个时候可以怎么做呢？
其实就可以采用ThreadLocal，采用ThreadLocal可以让监听器作为线程内的全局对象而存在，在线程内部只要通过get方法就可以获取到监听器。
而如果不采用ThreadLocal，那么我们能想到的可能是如下两种方法：第一种方法是将监听器通过参数的形式在函数调用栈中进行传递，
第二种方法就是将监听器作为静态变量供线程访问。上述这两种方法都是有局限性的。第一种方法的问题时当函数调用栈很深的时候，
通过函数参数来传递监听器对象这几乎是不可接受的，这会让程序的设计看起来很糟糕。第二种方法是可以接受的，但是这种状态是不具有可扩充性的，
比如如果同时有两个线程在执行，那么就需要提供两个静态的监听器对象，如果有10个线程在并发执行呢？提供10个静态的监听器对象？
这显然是不可思议的，而采用ThreadLocal每个监听器对象都在自己的线程内部存储，根据就不会有方法2的这种问题。

### ThreadLocal的基本操作

ThreadLocal类接口需要注意如下几个方法：

1.构造函数

    {% highlight java %}  
        /**
         * Creates a new thread-local variable.
         */
        public ThreadLocal() {}
    {% endhighlight %} 
    
构造函数内部什么也没做。    

2.initialValue函数

    {% highlight java %}  
    /**
     * Provides the initial value of this variable for the current thread.
     * The default implementation returns {@code null}.
     *
     * @return the initial value of the variable.
     */
    protected T initialValue() {
        return null;
    }
     {% endhighlight %} 
     
该函数在调用get函数的时候会第一次调用，但是如果一开始就调用了set函数，则该函数不会被调用。通常该函数只会被调用一次，
除非手动调用了remove函数之后又调用get函数，这种情况下，get函数中还是会调用initialValue函数。该函数是protected类型的，
很显然是建议在子类重载该函数的，所以通常该函数都会以匿名内部类的形式被重载，以指定初始值，比如：

    {% highlight java %}  
 ThreadLocal<Integer> b = new ThreadLocal<Integer>(){
        @Override
        protected Integer initialValue() {
            return 1;
        }
    };
    {% endhighlight %}   
    
3.get函数:public T get()
    
该函数用来获取与当前线程关联的ThreadLocal的值,如果当前线程没有该ThreadLocal的值，则调用initialValue函数获取初始值返回。
 
4.set函数:public void set(T value)

set函数用来设置当前线程的该ThreadLocal的值,设置当前线程的ThreadLocal的值为value。
 
5.remove函数:public void remove()

remove函数用来将当前线程的ThreadLocal绑定的值删除,在某些情况下需要手动调用该函数，防止内存泄露。

### ThreadLocal的使用

下面写一个demo来演示ThreadLocal的作用，用一个android的中多线程来演示，代码如下

    {% highlight java %}  
public class TestActivity extends Activity {
    private int a = 1 ;
    private ThreadLocal<Integer> b = new ThreadLocal<Integer>(){
        @Override
        protected Integer initialValue() {
            return 1;
        }
    };
    final Handler myHandler = new Handler() {
        @Override
        //重写handleMessage方法,根据msg中what的值判断是否执行后续操作
        public void handleMessage(Message msg) {
            Log.e(Thread.currentThread().getName(),"a的值是："+a+",   b的值是：" + b.get()) ;
        }
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test11);
        new Timer().schedule(new TimerTask() {
            @Override
            public void run() {
                a ++ ;
                b.set(a);
                Log.e(Thread.currentThread().getName(),"a的值是："+a+",   b的值是：" + b.get()) ;
                myHandler.sendEmptyMessage(0x123);
            }
        }, 0,1000);
    }
}
    {% endhighlight %}   
    
在activity中先初始两个变量a和b,都等于1，b使用ThreadLocal初始化，在子线程中每隔一秒改变a和b的值，然后分别输出：

![](https://raw.githubusercontent.com/mxn21/mxn21.github.io/master/public/img/img162.png)

可以看出a的值是两个线程共享的，多个线程可以同时改变a的值，如果不加上同步锁的情况下，这是线程不安全的。
而主线程中b的值始终是1，没有改变，而子线程中的b每次增加1，可见b的值并没有被共享，而是相互独立的。
 


### ThreadLocal的原理

### 总结

ThreadLocal的作用是提供线程内的局部变量，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或者组件之间一些公共变量的传递的复杂度。