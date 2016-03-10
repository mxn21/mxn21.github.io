---
layout: post
author: mxn
titile: 理解ThreadLocal
category: 技术博文
tag: android
---

ThreadLocal类用来提供线程内部的局部变量。这种变量在多线程环境下访问(通过get或set方法访问)时能保证各个线程里的变量相对独立于其他线程内的变量。

ThreadLocal不能使用原子类型，只能使用Object类型。
hreadLocal为每一个线程都提供了变量的副本， 使得每个线程在某一时间访问到的并不是同一个对象，
这样就隔离了多个线程对数据的数据共享。

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



### ThreadLocal的原理

ThreadLocal是如何做到为每一个线程维护变量的副本的呢？其实实现的思路很简单：在ThreadLocal类中有一个Map，
用于存储每一个线程的变量副本，Map中元素的键为线程对象，而值对应线程的变量副本。

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
 

### Thread同步机制的比较

ThreadLocal和线程同步机制相比有什么优势呢？ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。
在同步机制中，通过对象的锁机制保证同一时间只有一个线程访问变量。这时该变量是多个线程共享的，使用同步机制要求程序慎密地分析什么时候对变量进行读写，
什么时候需要锁定某个对象，什么时候释放对象锁等繁杂的问题，程序设计和编写难度相对较大。

而ThreadLocal则从另一个角度来解决多线程的并发访问。ThreadLocal会为每一个线程提供一个独立的变量副本，从而隔离了多个线程对数据的访问冲突。
因为每一个线程都拥有自己的变量副本，从而也就没有必要对该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封装进ThreadLocal。

概括起来说，对于多线程资源共享的问题，同步机制采用了“以时间换空间”的方式，而ThreadLocal采用了“以空间换时间”的方式。前者仅提供一份变量，
让不同的线程排队访问，而后者为每一个线程都提供了一份变量，因此可以同时访问而互不影响。

### 总结

ThreadLocal的作用是提供线程内的局部变量，这种变量在线程的生命周期内起作用，减少同一个线程内多个函数或者组件之间一些公共变量的传递的复杂度。