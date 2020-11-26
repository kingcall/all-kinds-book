- volatile 变量和 atomic 变量有什么不同？
这是个有趣的问题。首先，volatile 变量和 atomic 变量看起来很像，但功能却不一样。Volatile变量可以确保先行关系，即写操作会发生在后续的读操作之前, 但它并不能保证原子性。例如用volatile修饰count变量那么 count++ 操作就不是原子性的。而AtomicInteger类提供的atomic方法可以让这种操作具有原子性如getAndIncrement()方法会原子性 的进行增量操作把当前值加一，其它数据类型和引用变量也可以进行相似操作

### 不足
- CAS相对于其他锁，不会进行内核态操作，有着一些性能的提升。但同时引入自旋，当锁竞争较大的时候，自旋次数会增多。cpu资源会消耗很高。
- AtomicLong是通过无限循环不停的采取CAS的方法去设置内部的value，直到成功为止。那么当并发数比较多或出现更新热点时，就会导致CAS的失败机率变高，重试次数更多，越多的线程重试，CAS失败的机率越高，形成恶性循环，从而降低了效率。
。


## java8 中新引入的计数器
- Atomic*遇到的问题是，只能运用于低并发场景。因此LongAddr在这基础上引入了分段锁的概念
- LongAdder的原理就是降低对value更新的并发数，也就是将对单一value的变更压力分散到多个value值上，降低单个value的“热度”。
- 通过CAS乐观锁保证原子性，通过自旋保证当次修改的最终修改成功，通过降低锁粒度（多段锁）增加并发性能

### 详解
- 最初无竞争时，只更新base的值，当有多线程竞争时通过分段的思想，让不同的线程更新不同的段，最后把这些段相加就得到了完整的LongAdder存储的值。

![image-20201126163456432](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/16:34:57-image-20201126163456432.png)

- LongAdder通过base和cells数组来存储值；
- 不同的线程会hash到不同的cell上去更新，减少了竞争；
- LongAdder的性能非常高，最终会达到一种无竞争的状态；

### LongAdder
### LongAccumulator

### DoubleAdder
### DoubleAccumulator