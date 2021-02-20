[toc]
# 网络模型
## 网络传输模型
- 在一个 Flink Job 中，数据需要在不同的 task中进行交换，整个数据交换是由TaskManager 负责的，TaskManager 的网络组件首先从缓冲 buffer 中收集 records，然后再发送。Records 并不是一个一个被发送的，是积累一个批次再发送，batch技术可以更加高效的利用网络资源。

- 网络是以 batch 的形式来传输数据的，而每个 batch 都会带来额外的空间开销（header 等元数据）和时间开销（发送延迟、序列化反序列化延等），因此batchsize越大则传输的开销越小
- 但是这也会导致延时更高，因为数据需要在缓存中等待的时间越久对于实时类应用来说，我们通常希望延迟可以被限定在一个合理的范围内
- 因此业界大多数的做法是设置一个 batch timeout 来强制发送低于 batch size 的数据 batch，这通常需要额外设置设置一个线程来实现。

### TaskManager 
- 不同任务之间的每个（远程）网络连接将在 Flink 的网络栈中获得自己的 TCP 通道。但是，如果同一任务的不同子任务被安排到了同一个 TaskManager，则它们与同一个 TaskManager 的网络连接将被多路复用，并共享一个 TCP 信道以减少资源占用。在这个示例中是 A.1→B.3、A.1→B.4 以及 A.2→B.3 和 A.2→B.4 的情况
- TaskManager 传输数据时，不同的 TaskManager 上的两个 Subtask 间通常根据 key 的数量有多个 Channel，这些 Channel 会复用同一个 TaskManager 级别的 TCP 链接，并且共享接收端 Subtask 级别的 Buffer Pool

![image](https://note.youdao.com/yws/res/33810/FF609EE3307446F8A34B5F3C3F6B692A)


## Credit-based 数据流控制
### 背景(基于网络传输模型的)
- 在物理层 Flink 并不会为维护 Subtask 级别的 TCP 连接，**Flink 的 TCP 连接是 TaskManager 级别的**，相同两个 TaskManager 上不同的 Subtask 的数据传输会通过 Netty 实现复用和分用跑在同一条 TCP 连接上。
- 对于每个 Subtask来说，根据key的不同它可以输出数据到下游任意的 Subtask，因此 Subtask在内部会维护下游Subtask数目的发送队列，相对地，下游 Subtask 也会维护上游 Subtask 数目的接收队列。(队列的个数和下游Subtask的个数相等)
- 在接收端，每个 Channel 在初始阶段会被分配固定数量的 Exclusive Buffer，这些 Buffer 会被用于存储接受到的数据，交给 Operator 使用后再次被释放。
- Channel 接收端空闲的 Buffer 数量称为 Credit，Credit 会被定时同步给发送端被后者用于决定发送多少个 Buffer 的数据。
- 在流量较大时，Channel 的 Exclusive Buffer 可能会被写满，此时 Flink 会向 Buffer Pool 申请剩余的 Floating Buffer。这些 Floating Buffer 属于备用 Buffer，哪个 Channel 需要就去哪里。而在 Channel 发送端，一个 Subtask 所有的 Channel 会共享同一个 Buffer Pool，这边就没有区分 Exclusive Buffer 和 Floating Buffer。

### 存在的问题
- 这种实现的问题在于当某个Subtask出现阻塞时，该阻塞不仅会作用于该Subtask的Channel，还会误伤到这个 TaskManager上的其他Subtask，因为整个TCP连接都被阻塞了。
- 比如在图中，因为 Subtask 4 一个 Channel 没有空闲 Buffer，使用同一连接的其他 3 个 Channel 也无法通信。为了解决这个问题，Flink 自 1.5 版本**引入了 Credit-based 数据流控制为 TCP 连接提供更加细粒度的控制 **

![image-20210218193958715](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218193958715.png)

### Credit-based(基于信用的流量控制)

![image-20210218194029536](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218194029536.png)
- 在接受端的 Buffer 被划分为 Exclusive Buffer 和 Floating Buffer 两种，前者是固定分配到每条接受队列里面的，后者是在 Subtask 级别的 Buffer Pool 里供动态分配。
- 发送队列里的数据称为Blacklog，而接收队列里的Buffer称为Credit。接收器将缓存的可用性声明为发送方的信用（1缓存=1信用）
- Credit-Based 数据流控制的核心思想则是**根据接收端的空闲 Buffer 数（即 Credit）来控制发送速率，这和 TCP 的速率控制十分类似，不过是作用在应用层**。

#### 详解
- 假设当前发送队列有 5 个 Blacklog，而接收队列有 2 个空闲 Credit。
- 首先接收端会通知发送端可以发送2个Buffer，这个过程称**为AnnounceCredit**。随后发送端接收到请求后将 Channel Credit 设为 2，并发送 1 个 Buffer（随后 Channel Credit 减为 1），并将剩余4个Backlog的信息随着数据一起发给接收端，这个两个过程分为称为 **Send Buffer** 和 **Announce Blacklog Size**。
- 接收端收到 Backlog Size 之后会向 Buffer Pool 申请 Buffer 以将队列拓展至可以容纳 Backlog Size 的数据(这个是为了快速的处理，其实就是一次性处理)，但不一定能全部拿到。因为队列目前有一个空闲 Buffer，因此只需要向 Buffer Pool 申请 3 个 Buffer。假设 3 个 Buffer 都成功申请到，它们会成为 Unannounced Credit，并在下一轮请求中被 Announce。
- 一条 Channel 发送端的 Announced Credit 与 接收端的 Unannounced Credit 之和不小于 Blacklog Size 时，该 Channel 处于正常状态，否则处于反压状态

#### 总结
- Credit-based 数据流控制避免了阻塞 TCP 连接(不会因为一个channel 影响整个taskmanager )，使得资源可以更加充分地被利用，另外通过动态分配 Buffer 和拓展队列长度，可以更好地适应生产环境中的不断变化的数据分布及其带来的 Channel 状态抖动，也有利于缓减部分 Subtask 遇到错误或者处理速率降低造成的木桶效应。
- 然而这个新的机制也会引入额外的成本，即每传输一个 Buffer 要额外一轮 Announce Credit 请求来协商资源，不过从官方的测试来看，整体性能是有显著提升的。
- 信用机制提供了更直接的控制逻辑：如果接收器能力不足，其可用信用将减到 0，并阻止发送方将缓存转发到较底层的网络栈上。这样**只在这个逻辑信道上存在背压，并且不需要阻止从多路复用 TCP信道读取内容。因此，其他接收器在处理可用缓存时就不受影响了**
## 吞吐量和延迟之间做权衡
- Flink 天然支持流式处理，即每来一条数据就能处理一条，而不是像 Spark Streaming 一样，完全是微批处理。但是为了提高吞吐量，默认使用的Flink并不是每来一条数据就处理一条

### 触发刷新缓存的时机
#### Buffer 变满时
####  Buffer timeout 时
- Flink 在数据传输时，会把数据序列化成二进制然后写到 Buffer 中，当 Buffer 满了，需要 Flush（默认为32KiB，通过taskmanager.memory.segment-size设置）。
- 但是当流量低峰或者测试环节，可能1分钟都没有 32 KB的数据，就会导致1分钟内的数据都积攒在 Buffer 中不会发送到下游 Task 去处理，从而导致数据出现延迟，这并不是我们想看到的。所以 Flink 有一个 Buffer timeout 的策略，意思是当数据量比较少，Buffer 一直没有变满时，后台的 Output flusher 线程会强制地将 Buffer 中的数据 Flush 到下游。Flink 中默认 timeout 时间是 100ms，即：Buffer 中的数据要么变满时 Flush，要么最多等 100ms 也会 Flush 来保证数据不会出现很大的延迟。
- 当然这个可以通过 env.setBufferTimeout(timeoutMillis) 来控制超时时间。
##### timeoutMillis > 0 
- 表示最长等待 timeoutMillis 时间，就会flush
##### timeoutMillis = 0 
- 表示每条数据都会触发flush，直接将数据发送到下游，相当于没有Buffer了(避免设置为0，可能导致性能下降)
##### timeoutMillis = -1
- 表示只有等到 buffer满了或 CheckPoint的时候，才会flush。相当于取消了 timeout 策略


#### 特殊事件来临时CheckPoint 的 barrier 来临时
- 一些特殊的消息如果通过 RecordWriter 发送，也会触发立即 Flush 缓存的数据。
- 其中最重要的消息包括 Checkpoint barrier 以及 end-of-partition 事件，这些事件应该尽快被发送，而不应该等待 Buffer 被填满或者 Output flusher 的下一次 Flush。
- 当然如果出现反压，CheckPoint barrier 也会等待，不能发送到下游。


# 反压
- Flink 内部是基于 producer-consumer模型来进行消息传递的，Flink的反压设计也是基于这个模型。Flink 使用了高效有界的分布式阻塞队列，就像Java通用的阻塞队列（BlockingQueue）一样。下游消费者消费变慢，上游就会受到阻塞

## 背景
- 反压（backpressure）是实时计算应用开发中，特别是流式计算中，十分常见的问题。**反压意味着数据管道中某个节点成为瓶颈，处理速率跟不上上游发送数据的速率，而需要对上游进行限速**。
- 由于实时计算应用通常使用消息队列来进行生产端和消费端的解耦，消费端数据源是pull-based 的，所以反压通常是从某个节点传导至数据源并降低数据源（比如Kafkaconsumer）的摄入速率。
- 简单来说，Flink 拓扑中每个节点（Task）间的数据都以阻塞队列的方式传输，下游来不及消费导致队列被占满后，上游的生产也会被阻塞，最终导致数据源的摄入被阻塞。
- 对比来看，Spark Streaming的backpressure比较简单，主要是根据下游任务的执行情况等，来控制 Spark Streaming 上游的速率。
- Flink 的 back pressure机制不同，通过一定时间内stack     traces采样，监控阻塞的比率来确定背压的。

## 为什么需要反压
![image-20210218194055026](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218194055026.png)
- Producer 生产数据 2MB/s， Consumer 消费数据 1MB/s，Receive Buffer 只有 5MB，所以过了5秒后，接收端的 Receive Buffer 满了。
- 下游消费者会丢弃新到达的数据，因为下游消费者的缓冲区放不下
- 为了不丢弃数据，所以下游消费者的ReceiveBuffer持续扩张，最后耗尽消费者的内存，OOM，程序挂掉
- 第一种会把数据丢弃、第二种会把我们的应用程序挂掉。所以，该问题的解决方案不应该是下游 Receive Buffer 一直累积数据，而是上游 Producer 发现下游 Consumer 处理比较慢之后，应该在 Producer 端做出限流的策略，防止在下游 Consumer 端无限制的数据堆积。


### 静态限流的策略
- 静态限速的思想就是，提前已知下游 Consumer 的消费速率，然后通过在上游 Producer 端使用类似令牌桶的思想，限制 Producer 端生产数据的速率，从而控制上游 Producer 端向下游 Consumer 端发送数据的速率。
![image-20210218194105966](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218194105966.png)

#### 存在的问题
- 通常无法事先预估下游 Consumer 端能承受的最大速率
- 就算通过某种方式预估出下游Consumer端能承受的最大速率，下游应用程序也**可能会因为网络抖动、CPU共享竞争、内存紧张、IO阻塞等原因造成下游应用程序的吞吐量降低**，然后又会出现上面所说的下游接收区的ReceiveBuffer有限，上游一直有源源不断的数据发送到下游的问题，还是会造成下游要么丢数据，要么为了不丢数据 buffer 不断扩充导致下游 OOM的问题

### 动态限流
- 上游 Producer 端必须有一个限流的策略，且静态限流是不可靠的，于是就需要一个动态限流的策略
![image-20210218194116405](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218194116405.png)

## flink 中的反压
- 如果你看到一个task的backpressure告警（比如，high），这意味着生产数据比下游操作算子消费的速度快。Record的在你工作流的传输方向是向下游，比如从source到sink，而back pressure正好是沿着反方向，往上游传播。
- 如果没有正确处理反压力，可能会导致资源耗尽，甚至在最坏的情况下，数据丢失。
```
例子，一个工作流，只有source到sink两个步骤。假如你看到source端有个告警，这意味着sink消费数据速率慢于生产者的生产数据速率。Sink正在向上游进行back pressure。Sink 正在向 Source 施加反压。
```
### 原理(采样线程-线程采样)
- 是通过重复采样正在运行的tasks的tack trace样本数据来监控任务的。JobManager会针对你的job的tasks重复触发调用Thread.getStackTrace()。
- 如果样本数据显示任务线程卡在某个内部方法调用中（从网络堆栈请求缓冲区），则表示该任务存在背压
- 为了不使堆栈跟踪样本对TaskManager负载过高，每60秒会刷新采样数据

### 定位反压
- 前者比较容易上手，适合简单分析，后者则提供了更加丰富的信息，适合用于监控系统。因为反压会向上游传导，这两种方式都要求我们从Source节点到Sink的逐一排查，直到找到造成反压的根源原因

#### 通过 Flink Web UI 自带的反压监控面板
- Flink Web UI 的反压监控提供了 SubTask 级别的反压监控，原理是通过周期性对 Task 线程的栈信息采样，得到线程被阻塞在请求Buffer（意味着被下游队列阻塞）的频率来判断该节点是否处于反压状态。
- - 为了判断是否进行反压，jobmanager会每50ms触发100次stacktraces。Web界面中显示阻塞在内部方法调用的stacktraces占所有的百分比。
- 例如，0.01，代表着100次中有一次阻塞在内部调用。
> • OK: 0 <= Ratio <= 0.10
> • LOW: 0.10 < Ratio <= 0.5
> • HIGH: 0.5 < Ratio <= 1

#### 通过 Flink Task Metrics
![image-20210218194134432](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218194134432.png)
- 其中 inPoolUsage 等于 floatingBuffersUsage 与 exclusiveBuffersUsage 的总和。
- 如果一个 Subtask 的发送端 Buffer 占用率很高，则表明它被下游反压限速了
- 如果一个 Subtask 的接受端 Buffer 占用很高，则表明它将反压传导至上游。
- outPoolUsage 和 inPoolUsage 同为低或同为高分别表明当前 Subtask 正常或处于被下游反压
- 当 outPoolUsage 和 inPoolUsage表现不同时，这可能是出于反压传导的中间状态或者表明该 Subtask 就是反压的根源。如果一个 Subtask 的 outPoolUsage 是低，但其 inPoolUsage 是高，则表明它有可能是反压的根源。因为通常反压会传导至其上游，导致上游某些 Subtask 的 outPoolUsage 为高，我们可以根据这点来进一步判断。

### 反压配置
- web.backpressure.refresh-interval，统计数据被废弃重新刷新的时间（默认值：60000，1分钟）。
- web.backpressure.num-samples，用于确定背压的堆栈跟踪样本数（默认值：100）。
- web.backpressure.delay-between-samples，堆栈跟踪样本之间的延迟以确定背压（默认值：50，50ms)

### 反压的影响
- 反压并不会直接影响作业的可用性，它表明作业处于亚健康的状态，有潜在的性能瓶颈并可能导致更大的数据处理延迟。
- 通常来说，对于一些对延迟要求不太高或者数据量比较小的应用来说，反压的影响可能并不明显，然而对于规模比较大的 Flink 作业来说反压可能会导致严重的问题 
- 这两个影响对于生产环境的作业来说是十分危险的，因为 checkpoint 是保证数据一致性的关键，checkpoint 时间变长有可能导致 checkpoint 超时失败，而 state 大小同样可能拖慢 checkpoint 甚至导致 OOM （使用 Heap-based StateBackend）或者物理内存使用超出容器资源（使用 RocksDBStateBackend）的稳定性问题。

#### 影响checkpoint 
- 因为 checkpoint barrier 是不会越过普通数据的，数据处理被阻塞也会导致 checkpoint barrier 流经整个数据管道的时长变长，因而 checkpoint 总体时间（End to End Duration）变长。
- 这个时候可能会导致checkpoint 因为超时而失败，进而影响作业是容错，导致因失败恢复的数据链路边长

####  state 大小
- 为保证 EOS（Exactly-Once-Semantics，准确一次），对于有两个以上输入管道的 Operator，checkpoint barrier 需要对齐（Alignment），接受到较快的输入管道的 barrier 后，它后面数据会被缓存起来但不处理，直到较慢的输入管道的 barrier 也到达，**这些被缓存的数据会被放到state 里面，导致 checkpoint 变大**。

### 处理反压
- 产生反压的原因主要有，该节点的发送速率跟不上它的产生数据速率。这一般会发生在一条输入多条输出的 Operator（比如flatmap）；下游的节点接受速率较慢，通过反压机制限制了该节点的发送速率。
- 定位到反压节点后，分析造成原因的办法和我们分析一个普通程序的性能瓶颈的办法是十分类似的，可能还要更简单一点，因为我们要观察的主要是 Task Thread。

#### 下游GC
- TaskManager 的内存以及 GC 问题也可能会导致反压
- TaskManager JVM 各区内存不合理导致的频繁 Full GC 甚至失联
- 推荐可以通过给 TaskManager 启用 G1 垃圾回收器来优化 GC，并加上-XX:+PrintGCDetails 来打印 GC 日志的方式来观察 GC 的问题。

#### 流量高峰
- 数据源在发送数据的速度上达到峰值

#### 数据倾斜
- 这点我们可以通过 Web UI 各个 SubTask 的 Records Sent 和 Record Received 来确认
- 另外 Checkpoint detail里不同SubTask的State size也是一个分析数据倾斜的有用指标。

#### 代码执行效率
- 最有用的办法就是对 TaskManager 进行 CPU profile，从中我们可以分析到 Task Thread 是否跑满一个 CPU 核
- 如果是的话要分析 CPU 主要花费在哪些函数里面，比如我们生产环境中就偶尔遇到卡在 Regex 的用户函数（ReDoS）；
- 如果不是的话要看 TaskThread阻塞在哪里，可能是用户函数本身有些同步的调用
#### checkpoint
- 状态过大，或者 精确一次的语义也可能导致出现反压

#### 申请资源不足
- 当然，性能分析的结果也可能是正常的，只是作业申请的资源不足而导致了反压，这就通常要求拓展并行度。
- 值得一提的，在未来的版本Flink将会直接在WebUI提供JVM的CPU火焰图[5]，这将大大简化性能瓶颈的分析。

### 总结
- 反压是 Flink 应用运维中常见的问题，它不仅意味着性能瓶颈还可能导致作业的不稳定性。定位反压可以从 Web UI 的反压监控面板和 Task Metric 两者入手，前者方便简单分析，后者适合深入挖掘。
- 定位到反压节点后我们可以通过数据分布、CPU Profile和GC指标日志等手段来进一步分析反压背后的具体原因并进行针对性的优化。
- 通常来说，floatingBuffersUsage为高则表明反压正在传导至上游，而exclusiveBuffersUsage则表明了反压是**否存在倾斜**（floatingBuffersUsage高、exclusiveBuffersUsage低为有倾斜，因为少数 channel 占用了大部分的 Floating Buffer）。
- 基于网络的反压 metrics并不能定位到具体的Operator，只能定位到Task(往往是一个task-chain)
- 值得注意的是，反压有时是短暂的且影响不大，比如来自某个 Channel 的短暂网络延迟或者 TaskManager 的正常 GC，这种情况下我们可以不用处理

## spark 中的反压
### 背景
- Spark Streaming的backpressure出现的原因呢，我想大家应该都知道，是为了应对短期数据尖峰。
- SparkStreaming的backpressure是从spark1.5以后引入的**，在之前呢，只能通过限制最大消费速度（这个要人为压测预估）**，开启反压机制，即设置spark.streaming.backpressure.enabled为true，Spark Streaming会自动根据处理能力来调整输入速率，从而在流量高峰时仍能保证最大的吞吐和性能。

### 限流
> 对于基于Receiver 形式，我们可以通过配置 spark.streaming.receiver.maxRate 参数来限制每个 receiver 每秒最大可以接收的记录的数据

> 对于 Direct Approach 的数据接收，我们可以通过配置 spark.streaming.kafka.maxRatePerPartition参数来限制每次作业中每个Kafka分区最多读取的记录条数

- 这种限速的弊端很明显，比如假如我们后端处理能力超过了这个最大的限制，会导致资源浪费。需要对每个spark Streaming任务进行压测预估。

### 原理
1. RateController
2. RateEstimator
3. RateLimiter
- 这种机制呢实际上是基于自动控制理论的pid这个概念(速率估算器是根据这个实现的)

#### RateController
- 为了实现自动调节数据的传输速率，在原有的架构上新增了一个名为 RateController 的组件，这个组件继承自 StreamingListener，其监听所有作业的 onBatchCompleted 事件，并且基于 processingDelay、schedulingDelay、当前Batch处理的记录条数以及处理完成事件来估算出一个速率(调用RateEstimator)

```
override def onBatchCompleted(batchCompleted: StreamingListenerBatchCompleted) {
    val elements = batchCompleted.batchInfo.streamIdToInputInfo

    for {
      // 处理结束时间
      processingEnd <- batchCompleted.batchInfo.processingEndTime
      // 处理时间,即`processingEndTime` - `processingStartTime`
      workDelay <- batchCompleted.batchInfo.processingDelay
      // 在调度队列中的等待时间,即`processingStartTime` - `submissionTime`
      waitDelay <- batchCompleted.batchInfo.schedulingDelay
      // 当前批次处理的记录数
      elems <- elements.get(streamUID).map(_.numRecords)
    } computeAndPublish(processingEnd, elems, workDelay, waitDelay)
  }


private def computeAndPublish(time: Long, elems: Long, workDelay: Long, waitDelay: Long): Unit =
    Future[Unit] {
      // 根据处理时间、调度时间、当前Batch记录数，预估新速率
      val newRate = rateEstimator.compute(time, elems, workDelay, waitDelay)
      newRate.foreach { s =>
      // 设置新速率
        rateLimit.set(s.toLong)
      // 发布新速率
        publish(getLatestRate())
      }
    }
    
```

#### RateEstimator
- RateEstimator是速率估算器，主要用来估算**最大处理速率**，默认的在2.2之前版本中只支持PIDRateEstimator，在以后的版本可能会支持使用其他插件
- 这个速率主要用于更新流每秒能够处理的最大记录的条数。这样就可以实现处理能力好的话就会有一个较大的最大值，处理能力下降了就会生成一个较小的最大值。来保证Spark Streaming流畅运行

#### RateLimiter
- 以上这两个组件都是在Driver端用于更新最大速度的，而RateLimiter是用于接收到Driver的更新通知之后更新Executor的最大处理速率的组件。RateLimiter是一个抽象类，它并不是Spark本身实现的，而是**借助了第三方Google的GuavaRateLimiter来产生的**。
- 它实质上是一个限流器，也可以叫做令牌，如果Executor中task每秒计算的速度大于该值则阻塞，如果小于该值则通过，将流数据加入缓存中进行计算。这种机制也可以叫做令牌桶机制

![image-20210218194154648](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218194154648.png)
