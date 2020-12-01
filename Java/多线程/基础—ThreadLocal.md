[toc]
## 意义
- ThreadLocal是解决线程安全问题一个很好的思路，它通过为**每个线程提供一个独立的变量副本解决了变量并发访问的冲突问**题
> 也就是对于同一个ThreadLocal，每个线程通过get、set、remove接口操作只会影响自身线程的数据，不会干扰其他线程中的数据。
- 在很多情况下，ThreadLocal比直接使用synchronized同步机制解决线程安全问题更简单，更方便，且结果程序拥有更高的并发性。

## 场景
- 在Java的多线程编程中，为保证多个线程对共享变量的安全访问，通常会使用synchronized来保证同一时刻只有一个线程对共享变量进行操作。
- 这种情况下可以将类变量放到ThreadLocal类型的对象中，使变量在每个线程中都有独立拷贝，不会出现一个线程读取变量时而被另一个线程修改的现象。
- 最常见的ThreadLocal使用场景为用来解决数据库连接、Session管理等。
## 实现原理

ThreadLocal可以理解为线程本地变量，他会在每个线程都创建一个副本，那么在线程之间访问内部副本变量就行了，做到了线程之间互相隔离，相比于synchronized的做法是用空间来换时间。

ThreadLocal有一个静态内部类ThreadLocalMap，ThreadLocalMap又包含了一个Entry数组，Entry本身是一个弱引用，他的key是指向ThreadLocal的弱引用，Entry具备了保存key value键值对的能力。

弱引用的目的是为了防止内存泄露，如果是强引用那么ThreadLocal对象除非线程结束否则始终无法被回收，弱引用则会在下一次GC的时候被回收。

但是这样还是会存在内存泄露的问题，假如key和ThreadLocal对象被回收之后，entry中就存在key为null，但是value有值的entry对象，但是永远没办法被访问到，同样除非线程结束运行。

但是只要ThreadLocal使用恰当，在使用完之后调用remove方法删除Entry对象，实际上是不会出现这个问题的。

![image-20201201083834127](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/01/08:38:34-image-20201201083834127.png)