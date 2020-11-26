[toc]
## Disruptor
- 从功能上讲，它其实有点儿类似 Kafka。不过，和 Kafka 不同的是，Disruptor 是线程之间用于消息传递的队列。它在 Apache Storm、Camel、Log4j2等很多知名项目中都有广泛应用
- 它比 Java 中另外一个非常常用的内存消息队列 ArrayBlockingQueue（ABS）的性能，要高一个数量级，可以算得上是最快的内存消息队列了。它还因此获得过 Oracle 官方的 Duke 大奖
- 常见的内存队列往往采用循环队列来实现。这种实现方法，对于只有一个生产者和一个消费者的场景，已经足够了。但是，当存在多个生产者或者多个消费者的时候，单纯的循环队列的实现方式，就无法正确工作了
- 主要是因为，多个生产者在同时往队列中写入数据的时候，在某些情况下，会存在数据覆盖的问题。而多个消费者同时消费数据，在某些情况下，会存在消费重复数据的问题
- 实际上，不管架构设计还是产品设计，往往越简单的设计思路，越能更好地解决问题。正所谓“大道至简”，就是这个意思

### 基于循环队列的"生产者 - 消费者模型"
- 队列有两种实现思路。一种是基于链表实现的链式队列，另一种是基于数组实现的顺序队列。不同的需求背景下，我们会选择不同的实现方式。
- 实际上，相较于无界队列，有界队列的应用场景更加广泛。毕竟，我们的机器内存是有限的。而无界队列占用的内存数量是不可控的。
- 非循环的顺序队列在添加、删除数据的工程中，会涉及数据的搬移操作，导致性能变差。而循环队列正好可以解决这个数据搬移的问题，所以，性能更加好

### 单线程的生产消费模型
- 对于生产者和消费者之间操作的同步，我并没有用到线程相关的操作。而是采用了“当队列满了之后，生产者就轮训等待；当队列空了之后，消费者就轮训等待”这样的措施。

```

public class Queue {
  private Long[] data;
  private int size = 0, head = 0, tail = 0;
  public Queue(int size) {
    this.data = new Long[size];
    this.size = size;
  }

  public boolean add(Long element) {
    if ((tail + 1) % size == head) return false;
    data[tail] = element;
    tail = (tail + 1) % size;
    return true;
  }

  public Long poll() {
    if (head == tail) return null;
    long ret = data[head];
    head = (head + 1) % size;
    return ret;
  }
}

public class Producer {
  private Queue queue;
  public Producer(Queue queue) {
    this.queue = queue;
  }

  public void produce(Long data) throws InterruptedException {
    while (!queue.add(data)) {
      Thread.sleep(100);
    }
  }
}

public class Consumer {
  private Queue queue;
  public Consumer(Queue queue) {
    this.queue = queue;
  }

  public void comsume() throws InterruptedException {
    while (true) {
      Long data = queue.poll();
      if (data == null) {
        Thread.sleep(100);
      } else {
        // TODO:...消费数据的业务逻辑...
      }
    }
  }
}
```
### 基于加锁的并发"生产者 - 消费者模型"
- 只有一个生产者往队列中写数据，一个消费者从队列中读取数据，那上面的代码是没有问题的。但是，如果有多个生产者在并发地往队列中写入数据，或者多个消费者并发地从队列中消费数据，那上面的代码就不能正确工作了
    - 多个生产者写入的数据可能会互相覆盖
    - 多个消费者可能会读取重复的数据

#### 分析覆盖
- 两个线程同时往队列中添加数据，也就相当于两个线程同时执行类 Queue 中的 add() 函数。我们假设队列的大小 size 是 10，当前的 tail 指向下标 7，head指向下标3，也就是说，队列中还有空闲空间。
- 这个时候，线程 1 调用 add() 函数，往队列中添加一个值为 12 的数据；线程 2 调用 add() 函数，往队列中添加一个值为 15 的数据。在极端情况下，本来是往队列中添加了两个数据（12 和 15），最终可能只有一个数据添加成功，另一个数据会被覆盖

![image-20201126171928236](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:19:28-image-20201126171928236.png)
```
public boolean add(Long element) {
  if ((tail + 1) % size == head) return false;
  data[tail] = element;
  tail = (tail + 1) % size;
  return true;
}
```
- 从这段代码中，我们可以看到，第 3 行给 data[tail]赋值，然后第 4 行才给 tail 的值加一。赋值和tail加一两个操作，并非原子操作。
- 这就会导致这样的情况发生：当线程 1 和线程 2 同时执行 add() 函数的时候，线程 1 先执行完了第 3 行语句，将 data[7]（tail 等于 7）的值设置为 12。在线程 1 还未执行到第 4 行语句之前，也就是还未将 tail 加一之前，线程 2 执行了第 3 行语句，又将 data[7]的值设置为 15，也就是说，那线程 2 插入的数据覆盖了线程 1 插入的数据。原本应该插入两个数据（12 和 15）的，现在只插入了一个数据（15）。

![image-20201126171949445](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:19:50-image-20201126171949445.png)

#### 解决方案——加锁
- 最简单的处理方法就是给这段代码加锁，同一时间只允许一个线程执行 add() 函数。这就相当于将这段代码的执行，由并行改成了串行

#### 解决方案——CAS
- 用CAS（compare and swap，比较并交换）操作等减少加锁的粒度

### Disruptor 基于无锁的并发"生产者 - 消费者模型"
- 尽管 Disruptor的源码读起来很复杂，但是基本思想其实非常简单。实际上，它是换了一种队列和“生产者 - 消费者模型”的实现思路。
- 之前的实现思路中，队列只支持两个操作，添加数据和读取并移除数据，分别对应代码中的 add() 函数和 poll() 函数，而 Disruptor 采用了另一种实现思路。

#### 核心
- 对于生产者来说，它往队列中添加数据之前，先申请可用空闲存储单元，并且是批量地申请连续的 n 个（n≥1）存储单元。
- 当申请到这组连续的存储单元之后，后续往队列中添加元素，就可以不用加锁了，因为这组存储单元是这个线程独享的。
- 不过申请存储单元的过程是需要加锁的。
- 对于消费者来说，处理的过程跟生产者是类似的。它先去申请一批连续可读的存储单元（这个申请的过程也是需要加锁的），当申请到这批存储单元之后，后续的读取操作就可以不用加锁了。

#### 不足
- 不过，还有一个需要特别注意的地方，那就是，如果生产者 A 申请到了一组连续的存储单元，假设是下标为3到6的存储单元，生产者 B 紧跟着申请到了下标是7到9的存储单元，那在3到6没有完全写入数据之前，7 到 9 的数据是无法读取的。
- 这个也是 Disruptor 实现思路的一个弊端。

![image-20201126172108323](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/26/17:21:08-image-20201126172108323.png)

- Disruptor 采用的是 RingBuffer 和 AvailableBuffer 这两个结构，来实现我刚刚讲的功能。

## 扩展
### ID 生成器
- 实现一个 ID 生成器，可以为所有的用户表生成唯一的 ID 号。那现在问题是，如何设计一个高性能、支持并发的、能够生成全局唯一 ID 的 ID 生成

#### 不同步长
- 分库分表也可以使用自增主键，可以设置增加的步长。8台机器分别从1、2、3。。开始，步长8.从1开始的下一个id是9，与其他的不重复就可以了。

#### redis/zk
- redis或者zk应该也能生成自增主键，不过他们的写性能可能不能支持真正的高并发

#### 开放独立的id生成服务
- 最有名的算法应该是snowflake吧。snowflake的好处是基本有序，每秒钟可以生成很大的量，容易水平扩展。

#### Disruptor 
- 提前生成id存入disrupt，预估一下峰值时业务需要的id量，比如提前生成50万；
- 其实主要是借助这个思想，每个线程申请一个自增范围，当当前范围使用完了，就可以申请其他范围了。

