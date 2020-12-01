[toc]
## synchronized
> Java语言为了解决并发编程中存在的原子性、可见性和有序性问题，提供了一系列和并发处理相关的关键字，比如synchronized、volatile、final、concurren包等

- synchronized关键字在需要原子性、可见性和有序性这三种特性的时候都可以作为其中一种解决方案，看起来是“万能”的。的确，大部分并发控制操作都能使用synchronized来完成。
- 对于程序员来说，synchronized只是个关键字而已，用起来很简单。之所以我们可以在处理多线程问题时可以不用考虑太多，就是因为这个关键字帮我们屏蔽了很多细节。

### synchronized的用法
> 主要有两种用法，分别是同步方法和同步代码块。
> -对于同步方法，JVM采用ACC_SYNCHRONIZED标记符来实现同步。对于同步代码块。JVM采用monitorenter、monitorexit两个指令来实现同步。
>
> -  

```
public synchronized void doSth();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Hello World
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
      LineNumberTable:
        line 6: 0
        line 7: 8
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       9     0  this   Lthread/thread/sync/SynchronizedDemo;

  public void doSth1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: ldc           #5                  // class thread/thread/sync/SynchronizedDemo
         2: dup
         3: astore_1
         4: monitorenter
         5: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         8: ldc           #3                  // String Hello World
        10: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        13: aload_1
        14: monitorexit
        15: goto          23
        18: astore_2
        19: aload_1
        20: monitorexit
        21: aload_2
        22: athrow
        23: return

```
#### 方法级的同步
- 方法级的同步是隐式的。同步方法的常量池中会有一个ACC_SYNCHRONIZED标志。当某个线程要访问某个方法的时候，会检查是否有ACC_SYNCHRONIZED，如果有设置，则需要先获得监视器锁，然后开始执行方法，方法执行之后再释放监视器锁。
- 这时如果其他线程来请求执行方法，会因为无法获得监视器锁而被阻断住。
- 值得注意的是，如果在方法执行过程中，发生了异常，并且方法内部并没有处理该异常，**那么在异常被抛到方法外面之前监视器锁会被自动释放**
- 同步方法使用this锁。一个线程使用同步代码块(this明锁),另一个线程使用同步函数。如果两个线程抢票不能实现同步，那么会出现数据错误。


#### 代码块级同步
- 可以把执行monitorenter指令理解为加锁，执行monitorexit理解为释放锁。 每个对象维护着一个记录着被锁次数的计数器。
未被锁定的对象的该计数器为0，当一个线程获得锁（执行monitorenter）后，该计数器自增变为 1 ，当同一个线程再次获得该对象的锁的时候，计数器再次自增。当同一个线程释放锁（执行monitorexit指令）的时候，计数器再自减。当计数器为0的时候。锁将被释放，其他线程便可以获得锁。

#### 静态同步函数
- 方法上加上static关键字，使用synchronized 关键字修饰 或者使用类.class文件。
- 静态的同步函数使用的锁是  该函数所属字节码文件对象可以用 getClass方法获取，也可以用当前  类名.class 表示。

### synchronized与原子性
- 线程是CPU调度的基本单位。CPU有时间片的概念，会根据不同的调度算法进行线程调度。当一个线程获得时间片之后开始执行，在时间片耗尽之后，就会失去CPU使用权。所以在多线程场景下，由于时间片在线程间轮换，就会发生原子性问题。
- 通过monitorenter和monitorexit指令，可以保证被synchronized修饰的代码在同一时间只能被一个线程访问，**在锁未释放之前，无法被其他线程访问到**
- 因此，在Java中可以使用synchronized来保证方法和代码块内的操作是原子性的

> 线程1在执行monitorenter指令的时候，会对Monitor进行加锁，加锁后其他线程无法获得锁，除非线程1主动解锁。即使在执行过程中，由于某种原因，比如CPU时间片用完，线程1放弃了CPU，但是，他并没有进行解锁。而由于synchronized的锁是可重入的，下一个时间片还是只能被他自己获取到，还是会继续执行代码。直到所有代码执行完。这就保证了原子性。

### synchronized与可见性
- 为了保证可见性，有一条规则是这样的：对一个变量解锁之前，必须先把此变量同步回主存中。这样解锁后，后续线程就可以访问到被修改后的值。
- 线程加锁前，将清空工作内存中共享变量的值，从而使用共享变量时需要从主内存中重新读取最新的值。

### synchronized与有序性
- 有序性即程序执行的顺序按照代码的先后顺序执行
- CPU还可能对输入代码进行乱序执行，比如load->add->save 有可能被优化成load->save->add 。这就是可能存在有序性问题。
- 这里需要注意的是，synchronized是无法禁止指令重排和处理器优化的。也就是说，synchronized无法避免上述提到的问题。
- Java程序中天然的有序性可以总结为一句话：如果在本线程内观察，所有操作都是天然有序的。如果在一个线程中观察另一个线程，所有操作都是无序的。
- **由于synchronized修饰的代码，同一时间只能被同一线程访问。那么也就是单线程执行的。所以，可以保证其有序性**。

### synchronized与锁优化
- synchronized其实是借助Monitor实现的，在加锁时会调用objectMonitor的enter方法，解锁的时候会调用exit方法。事实上，只有在JDK1.6之前，**synchronized的实现才会直接调用ObjectMonitor的enter和exit，这种锁被称之为重量级锁**
- 在JDK1.6中出现对锁进行了很多的优化，进而出现**轻量级锁，偏向锁，锁消除，适应性自旋锁，锁粗化**(自旋锁在1.4就有，只不过默认的是关闭的，jdk1.6是默认开启的)，这些操作都是为了在线程之间更高效的共享数据 ，解决竞争问题。

## synchronized 锁优化

从JDK1.6版本之后，synchronized本身也在不断优化锁的机制，有些情况下他并不会是一个很重量级的锁了。优化机制包括自适应锁、自旋锁、锁消除、锁粗化、轻量级锁和偏向锁。

锁的状态从低到高依次为**无锁->偏向锁->轻量级锁->重量级锁**，升级的过程就是从低到高，降级在一定条件也是有可能发生的。

### 为什么要优化

synchronized监视器锁在互斥同步上对性能的影响很大。

Java的线程是映射到操作系统原生线程之上的，如果要阻塞或唤醒一个线程就需要操作系统的帮忙，这就要**从用户态转换到内核态，状态转换需要花费很多的处理器时间**。

所以频繁的通过Synchronized实现同步会严重影响到程序效率，这种锁机制也被称为重量级锁，为了减少重量级锁带来的性能开销，JDK对Synchronized进行了种种优化。



### synchronized 锁升级

- synchronized 锁升级原理：在锁对象的对象头里面有一个 threadid 字段，在第一次访问的时候 threadid 为空，jvm 让其持有**偏向锁**，并将 threadid 设置为其线程 id，再次进入的时候会先判断 threadid 是否与其线程 id 一致，如果一致则可以直接使用此对象，如果不一致，则**升级偏向锁为轻量级锁**，**通过自旋循环一定次数来获取锁**，执行一定次数之后，如果还没有正常获取到要使用的对象，此时就会**把锁从轻量级升级为重量级锁**，此过程就构成了 synchronized 锁的升级。
- 锁的升级的目的：锁升级是为了减低了锁带来的性能消耗。在Java6之后优化synchronized的实现方式，使用了偏向锁升级为轻量级锁再升级到重量级锁的方式，从而减低了锁带来的性能消耗。



**自旋锁**由于大部分时候，锁被占用的时间很短，共享变量的锁定时间也很短，所有没有必要挂起线程，用户态和内核态的来回上下文切换严重影响性能。自旋的概念就是让线程执行一个忙循环，可以理解为就是啥也不干，防止从用户态转入内核态，自旋锁可以通过设置-XX:+UseSpining来开启，自旋的默认次数是10次，可以使用-XX:PreBlockSpin设置。

**自适应锁**：自适应锁就是自适应的自旋锁，自旋的时间不是固定时间，而是由前一次在同一个锁上的自旋时间和锁的持有者状态来决定。

**偏向锁**：当线程访问同步块获取锁时，会在对象头和栈帧中的锁记录里存储偏向锁的线程ID，之后这个线程再次进入同步块时都不需要CAS来加锁和解锁了，偏向锁会永远偏向第一个获得锁的线程，如果后续没有其他线程获得过这个锁，持有锁的线程就永远不需要进行同步，反之，当有其他线程竞争偏向锁时，持有偏向锁的线程就会释放偏向锁。可以用过设置-XX:+UseBiasedLocking开启偏向锁。

**轻量级锁**：JVM的对象的对象头中包含有一些锁的标志位，代码进入同步块的时候，JVM将会使用CAS方式来尝试获取锁，如果更新成功则会把对象头中的状态位标记为轻量级锁，如果更新失败，当前线程就尝试自旋来获得锁。



#### 锁升级的过程



![image-20201130220149232](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/30/22:01:49-image-20201130220149232.png)

### 锁粗化

通常情况下，为了保证多线程间的有效并发，会要求每个线程持有锁的时间尽可能短，但是在某些情况下，一个程序对同一个锁不间断、高频地请求、同步与释放，会消耗掉一定的系统资源，因为锁的讲求、同步与释放本身会带来性能损耗，这样高频的锁请求就反而不利于系统性能的优化了，虽然单次同步操作的时间可能很短。**锁粗化就是告诉我们任何事情都有个度，有些情况下我们反而希望把很多次锁的请求合并成一个请求，以降低短时间内大量锁请求、同步、释放带来的性能损耗。**

其实就是说一般情况下我们希望synchronized  的代码块尽量小，但是如果一个方法中有很多这样加锁的代码块，这个时候我们可以适当的加大synchronized 的代码块，或者是给整个方法加锁

> 粗化指的是有很多操作都是对同一个对象进行加锁，就会把锁的同步范围扩展到整个操作序列之外。



```

public void doSomethingMethod(){
    synchronized(lock){
        //do some thing
    }
    //这是还有一些代码，做其它不需要同步的工作，但能很快执行完毕
    synchronized(lock){
        //do some thing
    }
     synchronized(lock){
        //do some thing
    }
     synchronized(lock){
        //do some thing
    }
     synchronized(lock){
        //do some thing
    }
     synchronized(lock){
        //do other thing
    }
    ... ...
}

```

上面这种情况我就可以给整个方法加锁，或者是合并代码块

```

public void doSomethingMethod(){
    //进行锁粗化：整合成一次锁请求、同步、释放
    synchronized(lock){
        //do some thing
        //做其它不需要同步但能很快执行完的工作
        //do other thing
    }

```



> 注意：这样做是有前提的，就是中间不需要同步的代码能够很快速地完成，如果不需要同步的代码需要花很长时间，就会导致同步块的执行需要花费很长的时间，这样做也就不合理了。


另一种需要锁粗化的极端的情况是

```

for(int i=0;i<size;i++){
    synchronized(lock){
    }
}
```

上面代码每次循环都会进行锁的请求、同步与释放，看起来貌似没什么问题，且在jdk内部会对这类代码锁的请求做一些优化，但是还不如把加锁代码写在循环体的外面，这样一次锁的请求就可以达到我们的要求，除非有特殊的需要：循环需要花很长时间，但其它线程等不起，要给它们执行的机会。

```
synchronized(lock){
    for(int i=0;i<size;i++){
    }
}
```



### 锁消除

锁削除是指虚拟机即时编译器在运行时，对一些代码上要求同步，但是被检测到不可能存在共享数据竞争的锁进行削除。锁削除的主要判定依据来源于逃逸分析的数据支持，如果判断到一段代码中，在堆上的所有数据都不会逃逸出去被其他线程访问到，那就可以把它们当作栈上数据对待，认为它们是线程私有的，同步加锁自然就无须进行。 

> 锁消除指的是JVM检测到一些同步的代码块，完全不存在数据竞争的场景，也就是不需要加锁，就会进行锁消除。

锁消除是发生在编译器级别的一种锁优化方式。有时候我们写的代码完全不需要加锁，却执行了加锁操作。

比如，StringBuffer类的append操作：

```
@Override
public synchronized StringBuffer append(String str) {
    toStringCache = null;
    super.append(str);
    return this;
}
```

从源码中可以看出，append方法用了synchronized关键词，它是线程安全的。但我们可能仅在线程内部把StringBuffer当作局部变量使用

```
public class Demo {
    public static void main(String[] args) {
        long start = System.currentTimeMillis();
        int size = 100000000;
        for (int i = 0; i < size; i++) {
            createStringBuffer("hello", "world");
        }
        long timeCost = System.currentTimeMillis() - start;
        System.out.println("createStringBuffer:" + timeCost + " ms");
    }
    public static String createStringBuffer(String str1, String str2) {
        StringBuffer sBuf = new StringBuffer();
        // append方法是同步操作
        sBuf.append(str1);
        sBuf.append(str2);
        return sBuf.toString();
    }
}

```



代码中createStringBuffer方法中的局部对象sBuf，就只在该方法内的作用域有效，不同线程同时调用createStringBuffer()方法时，都会创建不同的sBuf对象，因此此时的append操作若是使用同步操作，就是白白浪费的系统资源。

这时我们可以通过编译器将其优化，将锁消除，前提是java必须运行在server模式（server模式会比client模式作更多的优化），同时必须开启逃逸分析，关于更多逃逸分析，请看JVM 系列的 [《进阶—逃逸分析》]() 

-server -XX:+DoEscapeAnalysis -XX:+EliminateLocks,其中+DoEscapeAnalysis表示开启逃逸分析，+EliminateLocks表示锁消除。

所以这个时候编译器就根据逃逸分析的结果进行锁消除，认为这段代码不需要加锁，这个时候我们的代码被编异常class 文件后就没有进行加锁

> 逃逸分析：比如上面的代码，它要看sBuf是否可能逃出它的作用域？如果将sBuf作为方法的返回值进行返回，那么它在方法外部可能被当作一个全局对象使用，就有可能发生线程安全问题，这时就可以说sBuf这个对象发生逃逸了，因而不应将append操作的锁消除，但我们上面的代码没有发生锁逃逸，锁消除就可以带来一定的性能提升。



关闭锁消除的运行结果

![image-20201126162127002](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/16:21:30-16:21:27-image-20201126162127002.png)

开启锁消除的运行结果

![image-20201126162201481](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/16:22:04-16:22:02-image-20201126162201481.png)

## synchronized 总结

- 虽然说synchronized已经优化了很多，但是了lock 还是存在一定差距的
- 在性能上来说，如果竞争资源不激烈，两者的性能是差不多的，而当竞争资源非常激烈时（即有大量线程同时竞争），此时Lock的性能要远远优于synchronized。

### 对比lock

- Lock是一个接口，而synchronized是Java中的关键字，synchronized是内置的语言实现；
- synchronized在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而Lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能造成死锁现象，因此使用Lock时需要在finally块中释放锁
- **lock可以让等待锁的线程响应中断，而synchronized却不行，使用synchronized时，等待的线程会一直等待下去，不能够响应中断**
- 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到
- Lock可以提高多个线程进行读操作的效率。

#### 何时使用synchronized 
- 虽然java6 之后对synchronized 进行了优化，但是它依然是一个重量级锁，因为锁升级之后是没有办法降级的，例如打车软件如果使用synchronized，早高峰之后锁就会处于重量级锁，然后会影响整个系统的性能

## Monitor
- **无论是同步方法还是同步代码块都是基于监视器Monitor实现的**

### Monitor 定义
- 所有的Java对象是天生的Monitor，每一个Java对象都有成为Monitor的潜质，因为在Java的设计中，每一个Java对象自打娘胎里出来就带了一把看不见的锁，它叫做内部锁或者Monitor锁。
- 每个对象都存在着一个Monitor与之关联，对象与其Monitor之间的关系有存在多种实现方式，如Monitor可以与对象一起创建销毁。

### Moniter如何实现线程的同步
- 在Java虚拟机(HotSpot)中，monitor是由ObjectMonitor实现的（位于HotSpot虚拟机源码ObjectMonitor.hpp文件，C++实现的）。

### Monitor 的原理

#### 关键属性
- _owner：指向持有ObjectMonitor对象的线程
- _WaitSet：存放处于wait状态的线程队列
- _EntryList：存放处于等待锁block状态的线程队列
- _recursions：锁的重入次数
- _count：用来记录该线程获取锁的次数

#### 流程
- 线程T等待对象锁：_EntryList中加入T。
- 线程T获取对象锁：_EntryList移除T，_owner置为T，计数器_count- 加1。
- 线程T中锁对象调用wait()：_owner置为null，计数器_count减1，_WaitSet中加入T等待被唤醒。
- 持有对象锁的线程T执行完毕：复位变量的值，以便其他线程进入获- 取monitor
