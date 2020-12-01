[toc]
## 概论

相比synchronized的加锁方式来解决共享变量的内存可见性问题，**volatile就是更轻量的选择，他没有上下文切换的额外开销成本**。使用volatile声明的变量，可以确保值被更新的时候对其他线程立刻可见。volatile使用内存屏障来保证不会发生指令重排，解决了内存可见性的问题。

我们知道，线程都是从主内存中读取共享变量到工作内存来操作，完成之后再把结果写会主内存，但是这样就会带来可见性问题。举个例子，假设现在我们是两级缓存的双核CPU架构，包含L1、L2两级缓存。



线程A首先获取变量X的值，由于最初两级缓存都是空，所以直接从主内存中读取X，假设X初始值为0，线程A读取之后把X值都修改为1，同时写回主内存。这时候缓存和主内存的情况如下图。



![image-20201201082846083](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/01/08:28:46-image-20201201082846083.png)



线程B也同样读取变量X的值，由于L2缓存已经有缓存X=1，所以直接从L2缓存读取，之后线程B把X修改为2，同时写回L2和主内存。这时候的X值入下图所示。

那么线程A如果再想获取变量X的值，因为L1缓存已经有x=1了，所以这时候变量内存不可见问题就产生了，B修改为2的值对A来说没有感知。

![image-20201201083137632](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/01/08:31:38-image-20201201083137632.png)



那么，如果X变量用volatile修饰的话，当线程A再次读取变量X的话，CPU就会根据缓存一致性协议强制线程A重新从主内存加载最新的值到自己的工作内存，而不是直接用缓存中的值。

再来说内存屏障的问题，volatile修饰之后会加入不同的内存屏障来保证可见性的问题能正确执行。这里写的屏障基于书中提供的内容，但是实际上由于CPU架构不同，重排序的策略不同，提供的内存屏障也不一样，比如x86平台上，只有StoreLoad一种内存屏障。

1. StoreStore屏障，保证上面的普通写不和volatile写发生重排序
2. StoreLoad屏障，保证volatile写与后面可能的volatile读写不发生重排序
3. LoadLoad屏障，禁止volatile读与后面的普通读重排序
4. LoadStore屏障，禁止volatile读和后面的普通写重排序

![image-20201201083212787](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/01/08:32:13-image-20201201083212787.png)



## volatile

### 保证可见性
- volatile保证了不同线程对volatile修饰变量进行操作时的可见性。
- 一个线程修改volatile变量的值时，该变量的新值会立即刷新到主内存中，这个新值对其他线程来说是立即可见的。
- 一个线程读取volatile变量的值时，该变量在本地内存中缓存无效，需要到主内存中读取

#### 实现原理
- volatile内存屏障插入策略中有一条，“在每个volatile写操作的后面插入一个StoreLoad屏障”，这个指令主要完成了两件事情
>  将当前处理器缓存行的数据写回到系统内存。
> 这个写回内存的操作会使在其他CPU里缓存了该内存地址的数据无效。

### 保证有序性

```
1）当第二个操作是volatile写时，不管第一个操作是什么，都不能重排序。这个规则确保volatile写之前的操作不会被编译器重排序到volatile写之后。
2）当第一个操作是volatile读时，不管第二个操作是什么，都不能重排序。这个规则确保volatile读之后的操作不会被编译器重排序到volatile读之前。
3）当第一个操作是volatile写，第二个操作是volatile读时，不能重排序。
```
#### 实现原理
- 在每个volatile写操作的前面插入一个StoreStore屏障。
- 在每个volatile写操作的后面插入一个StoreLoad屏障。
- 在每个volatile读操作的后面插入一个LoadLoad屏障。
- 在每个volatile读操作的后面插入一个LoadStore屏障。
> Store：数据对其他处理器可见（即：刷新到内存中）
Load：让缓存中的数据失效，重新从主内存加载数据


## 应用场景
### 资源同步
```
boolean inited = false;// 初始化完成标志
//线程1:初始化完成，设置inited=true
new Thread() {
    public void run() {
        context = loadContext();   //语句1
        inited = true;             //语句2
    };
}.start();
//线程2:每隔1s检查是否完成初始化，初始化完成之后执行doSomething方法
new Thread() {
    public void run() {
        while(!inited){
          Thread.sleep(1000);
        }
        doSomething(context);
    };
}.start();
```
- 程1初始化配置，初始化完成，设置inited=true。线程2每隔1s检查是否完成初始化，初始化完成之后执行doSomething方法。
- 线程1中，语句1和语句2之间不存在数据依赖关系，JMM允许这种重排序。如果在程序执行过程中发生重排序，先执行语句2后执行语句1，会发生什么情况？当线程1先执行语句2时，配置并未加载，而inited=true设置初始化完成了。线程2执行时，读取到inited=true，直接执行doSomething方法，而此时配置未加载，程序执行就会有问题。

### 线程中断
```
boolean stop = false;// 是否中断线程1标志
//Tread1
new Thread() {
    public void run() {
        while(!stop) {
          doSomething();
        }
    };
}.start();
//Tread2
new Thread() {
    public void run() {
        stop = true;
    };
}.start();
```
- Tread2设置stop=true时，Tread1读取到stop=true，Tread1中断执行。
