[toc]
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
