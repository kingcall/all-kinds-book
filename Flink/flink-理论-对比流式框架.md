[toc]
## 概论
- Apache Storm、Apache Spark 和 Apache Flink 都是开源社区中非常活跃的分布式计算平台，在很多公司可能同时使用着其中两种甚至三种。
- 对于实时计算来说，Storm 与 Flink 的底层计算引擎是基于流的，本质上是一条一条的数据进行处理，且处理的模式是流水线模式，即所有的处理进程同时存在，数据在这些进程之间流动处理。
- 而 Spark 是基于批量数据的处理，即一小批一小批的数据进行处理，且处理的逻辑在一批数据准备好之后才会进行计算。在本文中，我们把同样基于流处理的 Storm 和 Flink 拿来做对比测试分析。
## 指标
- 吞吐表示单位时间内所能处理的数据量，是可以通过增大并发来提高的。
- 延迟代表处理一条数据所需要的时间，与吞吐量成反比关系。

![image-20210218190328185](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218190328185.png)
- 在我们设计计算逻辑时，首先考虑一下流处理的计算模型。上图是一个简单的流计算模型，在 Source 中将数据取出，发往下游 Task ，并在 Task 中进行处理，最后输出。
- 对于这样的一个计算模型，延迟时间由三部分组成：数据传输时间、 Task 计算时间和数据排队时间。我们假设资源足够，数据不用排队。则延迟时间就只由数据传输时间和 Task 计算时间组成。而在Task中处理所需要的时间与用户的逻辑息息相关，
- 所以对于一个计算平台来说，数据传输的时间才更能反映这个计算平台的能力。因此，我们在设计测试 Case 时，为了更好的体现出数据传输的能力，Task 中没有设计任何计算逻辑。
### 延迟(数据传输时间)
- 数据源

![image-20210218190034891](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218190034891.png)
- 传输模型
- 对于数据传输方式，可以分为两种：进程间的数据传输和进程内的数据传输。

![image-20210218190205244](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218190205244.png)

- 进程间的数据传输是指这条数据会经过序列化、网络传输和反序列化三个步骤。在 Flink 中，2个处理逻辑分布在不同的TaskManager上，这两个处理逻辑之间的数据传输就可以叫做进程间的数据传输。
    -   Flink 网络传输是采用的 Netty 技术。
    -   在 Storm 中，进程间的数据传输是worker之间的数据传输。早版本的storm网络传输使用的 ZeroMQ，现在也改成了 Netty。
- 进程内的数据传输是指两个处理逻辑在同一个进程中。在 Flink 中，这两个处理逻辑被 Chain 在了一起，在一个线程中通过方法调用传参的形式进程数据传输。在 Storm 中，两个处理逻辑变成了两个线程，通过一个共享的队列进行数据传输。
#### 对比结果

![image-20210218190225151](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218190225151.png)
- 可以看到进程内的数据传输是进程间的数据传输的3.8倍。是否开启 checkpoint 机制对 Flink 的吞吐影响并不大。因此我们在使用 Flink 时，进来使用进程内的传输，也就是尽可能的让算子可以 Chain 起来。
- 大家知道我们在 Flink 代码时一定会创建一个 env，调用 env 的 disableOperatorChainning() 方法会使得所有的算子都无法 chain 起来。我们一般是在 debug 的时候回调用这个方法，方便调试问题。

![image-20210218190252285](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218190252285.png)
- 如果允许 Chain 的情况下，上图中 Source 和 mapFunction 就会 Chain 起来，放在一个 Task 中计算。反之，如果不允许 Chain，则会放到两个 Task 中。

### 可靠性

- Storm 和 Flink 都有各自的可靠性机制。在 Storm 中，使用 ACK 机制来保证数据的可靠性。而在 Flink 中是通过 checkpoint 机制来保证的，这是来源于 chandy-lamport 算法
- 事实上 Exactly-once 可靠性的保证跟处理的逻辑和结果输出的设计有关。比如结果要输出到kafka中，而输出到kafka的数据无法回滚，这就无法保证 Exactly-once。我们在测试的时候选用的 at-least-once 语义的可靠性和不保证可靠性两种策略进行测试。

## Flink vs Spark
### 调度计算和调度数据
- 对于任何一个分布式计算框架而言，如果“数据”和“计算”不在同一个节点上，那么它们中必须有一个需要移动到另一个所在的节点。如果把计算调度到数据所在的节点，那就是“调度计算”，反之则是“调度数据”；

#### spark
- Spark 的核心数据结构RDD包含了几个关键信息，包括数据的分片（partitions)、依赖（dependencies)等，其中还有一个用于优化执行的信息，就是分片的“preferred locations”
- 这个信息提供了该分片数据的位置信息，即所在的节点；Spark 在调度该分片的计算的时候，会尽量把该分片的计算调度到数据所在的节点，从而提高计算效率。
- 比如对于 KafkaRDD，该方法返回的就是 topic partition 的 leader 节点信息：
- "调度计算"的方法在批处理中有很大的优势，因为“计算”相比于“数据”来讲一般信息量比较小，如果“计算”可以在“数据”所在的节点执行的话，会省去大量网络传输，节省带宽的同时提高了计算效率。但是在流式计算中，以 Spark Streaming 的调度方法为例，由于需要频繁的调度”计算“，则会有一些效率上的损耗。
- 每次”计算“的调度都是要消耗一些时间的，比如“计算”信息的序列化 → 传输 → 反序列化 → 初始化相关资源 → 计算执行→执行完的清理和结果上报等，这些都是一些“损耗”
- 另外，用户的计算中一般会有一些资源的初始化逻辑，比如初始化外部系统的客户端（类似于 Kafka Producer 或 Consumer)；每次计算的重复调度容易导致这些资源的重复初始化，需要用户对执行逻辑有一定的理解，才能合理地初始化资源，避免资源的重复创建；
```
rdd.foreachPartition { partitionOfRecords =>
// ConnectionPool is a static, lazily initialized pool of connections
    val connection = ConnectionPool.getConnection()
    partitionOfRecords.foreach(record => connection.send(record))
    ConnectionPool.returnConnection(connection)  // return to the pool for future reuse
  }
}
```
- 需要指出的是，即使用户代码层面合理的使用了连接池，由于同一个"计算"逻辑不一定调度到同一个计算节点，还是可能会出现在不同计算节点上重新创建连接的情况

#### flink(storm)
- Flink 和 Storm 类似，都是通过“调度数据”来完成计算的，也就是“计算逻辑”初始化并启动后，如果没有异常会一直执行，源源不断地消费上游的数据，处理后发送到下游；有点像工厂里的流水线，货物在传送带上一直传递，每个工人专注完成自己的处理逻辑即可。
- 虽然“调度数据”和“调度计算”有各自的优势，但是在流式计算的实际生产场景中，“调度计算”很可能“有力使不出来”；比如一般流式计算都是消费消息队列 Kafka或 Talos 的数据进行处理，而实际生产环境中为了保证消息队列的低延迟和易维护，一般不会和计算节点（比如 Yarn 服务的节点）混布，而是有各自的机器（不过很多时候是在同一机房）；所以无论是 Spark 还是 Flink，都无法避免消息队列数据的跨网络传输。所以从实际使用体验上讲，Flink 的调度数据模式，显然更容易减少损耗，提高计算效率，同时在使用上更符合用户“直觉”，不易出现重复创建资源的情况。
- Spark Streaming 的“调度计算”模式，对于处理计算系统中的“慢节点”或“异常节点”有天然的优势。比如如果 Yarn 集群中有一台节点磁盘存在异常，导致计算不停地失败，Spark 可以通过 blacklist 机制停止调度计算到该节点，从而保证整个作业的稳定性。或者有一台计算节点的 CPU Load 偏高，导致处理比较慢，Spark 也可以通过 speculation 机制及时把同一计算调度到其他节点，避免慢节点拖慢整个作业；而以上特性在 Flink 中都是缺失的
