[toc]
## 基于窗口的join
- Windows类型的join都是利用window的机制，先将数据缓存在Window State中，当窗口触发计算时，执行join操作；

### Tumbling Window Join

### Sliding Window Join

### Session Widnow Join

## Interval Join
- 在流与流的join中，与其他window join相比，window中的关联通常是两个流中对应的window中的消息可以发生关联， 不能跨window
- Interval Join则没有window的概念，直接用时间戳作为关联的条件，更具表达力。由于流消息的无限性以及消息乱序的影响，本应关联上的消息可能进入处理系统的时间有较大差异
- interval join也是利用state存储数据再处理，区别在于state中的数据有失效机制，依靠数据触发数据清理；

```
1. 等值条件如 a.id = b.id
2. 时间戳范围条件
    a.timestamp ∈ [b.timestamp + lowerBound;b.timestamp + upperBound] 
    b.timestamp + lowerBound <= a.timestamp and a.timestamp <= b.timestamp +upperBound
```
### 实现原理
- 两条流数据缓存在内部 State 中，任意一数据到达，获取对面流相应时间范围数据，执行 joinFunction 进行 Join。随着时间的推进，State 中两条流相应时间范围的数据会被清理。

#### connect操作
- intervaljoin首先会将两个KeyedStream 进行connect操作得到一个ConnectedStreams， ConnectedStreams表示的是连接两个数据流，并且这两个数据流之前可以实现状态共享, 对于intervaljoin 来说就是两个流相同key的数据可以相互访问

#### ConnectedStreams之上进行IntervalJoinOperator算子操作
-  定义了两个MapState<Long, List<BufferEntry<T1>>>类型的状态对象，分别用来存储两个流的数据，其中Long对应数据的时间戳，List<BufferEntry<T1>>对应相同时间戳的数据、
-   包含processElement1、processElement2两个方法，这两个方法都会调用processElement方法，真正数据处理的地方
- 判断延时，数据时间小于当前的watermark值认为数据延时，则不处理
- 将数据添加到对应的MapState<Long, List<BufferEntry<T1>>>缓存状态中，key为数据的时间
循环遍历另外一个状态,如果满足ourTimestamp + relativeLowerBound <=timestamp<= ourTimestamp + relativeUpperBound , 则将数据输出给ProcessJoinFunction调用，ourTimestamp表示流入的数据时间，timestamp表示对应join的数据时间
- 注册一个数据清理时间方法，会调用onEventTime方法清理对应状态数据。对于例子中orderStream比addressStream早到1到5秒，那么orderStream的数据清理时间就是5秒之后，也就是orderStream.time+5,当watermark大于该时间就需要清理，对于addressStream是晚来的数据不需要等待，当watermark大于数据时间就可以清理掉。




## connet 的意义
- connet 没有匹配条件,分别处理两个流的元素，在此基础上可以实现 join 和 cogroup
- 一个connect操作是更普遍的则连接操作

## cogroup
- 侧重于group是对同一个key 的两个集合进行操作 ,cogroup和join 有类似的功能都是找到匹配的，但是cogroup找到的是**两个集合**
- 它在一个流/数据集中没有找到与另一个匹配的数据集还是会输出。
- 如果我们的逻辑不仅仅是对一条record做处理，而是要与上一record或更复杂的判断与比较，甚至是对结果排序

### 外连接的实现
- Flink 中DataStream 只提供了inner join 的实现，并未提供left join 与 right join 的实现，那么同样可以通过CoGroup来实现这两种join,以left join 为例，处理逻辑在CoGroupFunction中
## join
- join 可以看做是cogroup的特例，输出的是join 的结果(pair),只能在window中使用

  ![image-20210202095012407](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202095012407.png)
```
 override def join(in1: (Int, String, Long), in2: (Int, String, Long)): (Int, String, String) = {
      (in1._1, in1._2, in2._2)
    }
```
- Interval Join 也一样，收到的是元素
```
public abstract void processElement(IN1 left, IN2 right, Context ctx, Collector<OUT> out) throws Exception;
```

### 原理
1. 双流上的数据在同一个key的会被分别分配到同一个window窗口的左右两个篮子里
2. 当window结束的时候，会对左右篮子进行笛卡尔积从而得到每一对pair，**对每一对pair应用 JoinFunction**,所以函数的参数是单个对象，而不是集合
3. 因为目前join窗口的双流数据都是被缓存在内存中的，如果某个key对应的数据太多导致jvm OOM(数据倾斜是常态)-


## SQL Join
### Global Join
- Global Join表示全局join, 也可以称为无限流join, 由于没有时间限制，任何时候流入的数据都可以被关联上，支持inner join、left join、right join、full join 连接语法

### Time-windowed Join
