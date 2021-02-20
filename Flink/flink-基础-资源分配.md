[toc]

- 在 Operator 和 Task 中间的 Chaining 是指如何用 Operator 组成 Task 。
- 在 Task 和 Job 之间的 Slot Sharing 是指多个 Task 如何共享一个 Slot 资源，这种情况不会发生在跨作业的情况中。在 Flink Cluster 和 Job 之间的 Slot Allocation 是指 Flink Cluster 中的 Slot 是怎样分配给不同的 Job 。

## Operator
- 算子是最基本的数据处理单元
## Task
- Flink Runtime 中真正去进行调度的最小单位
由一系列算子链式组合而成（chained operators）

> 如果两个 Operator 属于同一个 Task，那么不会出现一个 Operator 已经开始运行另一个 Operator 还没被调度的情况

## task-chain
- 每个算子的一个并行度实例就是一个subtask-在这里为了区分暂时叫做substask。
- 由于flink的taskmanager运行task的时候是每个task采用一个单独的线程，这就会带来很多线程切换开销，进而影响吞吐量。
- 为了减轻这种情况，flink进行了优化，也即对subtask进行链式操作，链式操作结束之后得到的task，再作为一个调度执行单元，放到一个线程里执行,它能减少线程之间的切换，减少消息的序列化/反序列化，减少数据在缓冲区的交换，减少了延迟的同时提高整体的吞吐量。

> Flink即使是在同一个TaskManager的不同task(不同线程)进行数据传输，也不会产生网络通信。

### 详解
![image-20210202091140875](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202091140875.png)
- 图中假设是source/map的并行度都是2，keyby/window/apply的并行度也都是2，sink的是1，总共task有七个，最终需要七个线程,但是优化之后只需要5个线程

#### 数据流（逻辑视图）

- 创建Source（并行度设置为1）读取数据源，数据经过FlatMap（并行度设置为2）做转换操作，然后数据经过Key Agg（并行度设置为2）做聚合操作，最后数据经过Sink（并行度设置为2）将数据输出。

#### 数据流（并行化视图）
- 并行度为1的Source读取数据源，然后FlatMap并行度为2读取数据源进行转化操作，然后数据经过Shuffle交给并行度为2的Key Agg进行聚合操作，然后并行度为2的Sink将数据输出，未优化前的task总和为7。

#### 数据流（优化后视图）
- 并行度为1的Source读取数据源，然后FlatMap并行度为2读取数据源进行转化操作，然后数据经过Shuffle交给Key Agg进行聚合操作，此时Key Agg和Sink操作合并为一个task（注意：将KeyAgg和Sink两个operator进行了合并，因为这两个合并后并不会改变整体的拓扑结构），它们一起的并行度为2，数据经过Key Agg和Sink之后将数据输出，优化后的task总和为5.

- flink允许如果任务是不同的task的时候，允许任务共享slot，当然，前提是必须在同一个job内部。结果就是，每个slot可以执行job的一整个pipeline


### chain 条件
- 上下游的并行度一致
- 下游节点的入度为1 （也就是说下游节点没有来自其他节点的输入）
- 上下游节点都在同一个 slot group 中（下面会解释 slot group）
- 下游节点的 chain 策略为ALWAYS（可以与上下游链接，map、flatmap、filter等默认是ALWAYS）
- 上游节点的 chain 策略为 ALWAYS 或 HEAD（只能与下游链接，不能与上游链接，Source默认是HEAD）
- 两个节点间数据分区方式是 forward（参考理解数据流的分区）
- 用户没有禁用 chain

### 优势
- **它能减少线程之间的切换，减少消息的序列化/反序列化，减少数据在缓冲区的交换，减少了延迟的同时提高整体的吞吐量**

### 编程改变OperatorChain行为
- 通过在DataStream的operator后面（如someStream.map(..))调用startNewChain()来指示从该operator开始一个新的chain（与前面截断，不会被chain到前面）
- 可以调用disableChaining()来指示该operator不参与chaining（不会与前后的operator chain一起）
- 可以通过调用StreamExecutionEnvironment.disableOperatorChaining()来全局禁用chaining。
- 可以设置Slot group，例如someStream.filter(...).slotSharingGroup(“name”)。可以通过调整并行度，来调整Operator chain。

### 实现原理
![image-20210202091206213](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202091206213.png)
- Flink内部是通过OperatorChain这个类来将多个operator链在一起形成一个新的operator。
- OperatorChain形成的框框就像一个黑盒，Flink 无需知道黑盒中有多少个ChainOperator、数据在chain内部是怎么流动的，只需要将input数据交给 HeadOperator 就可以了，这就使得OperatorChain在行为上与普通的operator无差别，上面的OperaotrChain就可以看做是一个入度为1，出度为2的operator。
- 所以在实现中，对外可见的只有HeadOperator，以及与外部连通的实线输出，这些输出对应了JobGraph中的JobEdge，在底层通过RecordWriterOutput来实现。另外，框中的虚线是operator chain内部的数据流，这个流内的数据不会经过序列化/反序列化、网络传输，而是直接将消息对象传递给下游的 ChainOperator 处理，这是性能提升的关键点，在底层是通过 ChainingOutput 实现的

### 总结
- 没有任务链的情况，算子都是由独立的线程执行单独的task进行计算，数据被序列化后在线程间传输，而且也有可能用到TaskManager的网络缓冲(如果task在不同的worker里执行)，这样可能会降低程序执行效率。
- 当然，在某些场景下，没有任务链的执行效率可能比有任务链的执行效率还要高。
- Flink任务链功能和Spark Stage划分机制有异曲同工之处，都是为了实现数据的本地化处理。


## Task Slot
- slot在flink里面可以认为是**资源组/或者是资源分配单位**，Flink是通过将任务分成子任务并且**将这些子任务()分配到slot来并行执行程序**。
- TaskManager 是一个JVM进程，并会以独立的线程来执行一个task或多个subtask,**为了控制一个TaskManager能接受多少个task，Flink 提出了 Task Slot 的概念**。
- TaskManager会将自己节点上管理的资源分为不同的Slot：固定大小的资源子集。这样就避免了不同Job的Task互相竞争内存资源，但是需要主要的是，**Slot只会做内存的隔离。没有做CPU的隔离**(也就是说还会有影响的)

> 这个slot对资源的隔离**仅仅是对内存进行隔离，策略是均分**，比如taskmanager的管理内存是3GB，假如有两个个slot，那么每个slot就仅仅有1.5GB内存可用。
- 运行在同一个JVM 上的task 可以共享TCP连接，减少网络传输(在 Flink 集群中，一个 TaskManger 就是一个 JVM 进程，并且会用独立的线程来执行 task)

### CPU 和 slot 的关系
- 插槽的数量通常与每个TaskManager的可用CPU内核数成比例。一般情况下你的slot数是你每个TM的cpu的核数。
- 注意，CPU资源并不是每个slot所独享的，而是共享的。每个TaskManager拥有几个slot，就代表这个TaskManager能够提供的并发能力就是几，但实际的并行度也有可能小于这个数字。


### 任务和slot 的关系
![image-20210202091239913](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202091239913.png)


### IO密集型任务
- Flink 需要 IO 密集是很麻烦的问题，因为 Flink 的资源调度体系内有内存和 CPU，但 IO 单位未做统一管理。
- 当某一个作业对 IO 有强烈的需求时，需要分配很多以CPU或内存为单位的资源，且未必能够很好的满足 IO 的扩展

### task 隔离
- **通过调整 taskslot的数量，用户可以定义task之间是如何相互隔离的**。每个 TaskManager 有一个 slot，也就意味着每个 task 运行在独立的 JVM中。可达到应用程序独立部署、资源隔离的目的，防止异常的应用干扰其他无关的应用。
- 每个 TaskManager 有多个 slot 的话，也就是说多个 task 运行在同一个 JVM中。而在同一个JVM进程中的task，可以**共享TCP连接（基于多路复用）和心跳消息，可以减少数据的网络传输，也能共享一些数据结构、数据集合，一定程度上减少了每个 task 的消耗**


#### 隐患
- 需要注意的是，单个task在运行时可能会把整个work的内存占满，**导致一个异常的任务可能会把整个TM进程kill掉，这样运行在之上的其他任务也都被kill掉了**。



### 共享slot
- 默认情况下，Flink允许subtasks共享slot，**条件是它们都来自同一个Job的不同task的subtask**(Flink 允许子任务共享 Slot，即使它们是不同 task 的 subtask，只要它们来自相同的
job)结果可能一个slot持有该job的整个pipeline，**其实这个时候可以看出来共享的前提是pipline 或者说是taskchain**
> 这样可以避免多个操作在不同线程执行带来的上下文切换损失，并且可以直接在一个jvm中共享数据，这个思路可以说是一种新的优化思路

#### 实现原理
- SlotSharingGroup是Flink中用来实现slot共享的类，它尽可能地让subtasks共享一个slot。相应的，还有一个 CoLocationGroup 类用来强制将 subtasks 放到同一个 slot 中。
- CoLocationGroup主要用于迭代流中，用来保证迭代头与迭代尾的第i个subtask能被调度到同一个TaskManager上。
- **默认情况下，所有的operator都属于默认的共享组default**，也就是说默认情况下所有的operator都是可以共享一个slot的。
- 而当所有inputoperators具有相同的slot共享组时，该operator会继承这个共享组。最后，为了防止不合理的共享，用户也能通过API来强制指定operator的共享组，比如：someStream.filter(...).slotSharingGroup("group1");就强制指定了filter的slot共享组为group1。

#### 意义
- Flink 集群所需的**taskslots数与job中最高的并行度一致**。也就是说我们不需要再去计算一个程序总共会起多少个task了。
- 更容易获得更充分的资源利用。如果没有slot共享，那么非密集型操作source/flatmap就会占用同密集型操作keyAggregation/sink 一样多的资源。如果有slot共享，将基线的2个并行度增加到6个，能充分利用slot资源，同时保证每个TaskManager能平均分配到重的subtasks


### slot group
- 为了防止同一个slot包含太多的task，Flink提供了资源组(group)的概念。group就是对operator进行分组，同一group的不同operator task可以共享同一个slot。默认所有operator属于同一个组"default"，也就是所有operatortask可以共享一个slot。我们可以通过slotSharingGroup()为不同的operator设置不同的group。

#### 实现原理
![image-20210202091321200](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202091321200.png)
- 抽象类Slot定义了该槽位属于哪个TaskManager（instance）的第几个槽位（slotNumber），属于哪个Job（jobID）等信息。
- 最简单的情况下，一个slot只持有一个task，也就是SimpleSlot的实现。
- 复杂点的情况，一个slot能共享给多个task使用，也就是SharedSlot的实现。SharedSlot能包含其他的SharedSlot，也能包含SimpleSlot。所以一个SharedSlot能定义出一棵slots树

###### task 分配
- 同一个operator的各个subtask是不能呆在同一个SharedSlot中的，例如FlatMap[1]和FlatMap[2]是不能在同一个SharedSlot中的
- Flink是按照拓扑顺序从Source一个个调度到Sink的。例如WordCount（Source并行度为1，其他并行度为2），那么调度的顺序依次是：Source -> FlatMap[1] -> FlatMap[2] -> KeyAgg->Sink[1] -> KeyAgg->Sink[2]。

![image-20210202091331908](/Users/liuwenqiang/Library/Application%20Support/typora-user-images/image-20210202091331908.png)

1. 为Source分配slot。首先，我们从TaskManager1中分配出一个SharedSlot。并从SharedSlot中为Source分配出一个SimpleSlot。如上图中的①和②。
1. 为FlatMap[1]分配slot。目前已经有一个SharedSlot，则从该SharedSlot中分配出一个SimpleSlot用来部署FlatMap[1]。如上图中的③。
1. 为FlatMap[2]分配slot。由于TaskManager1的SharedSlot中已经有同operator的FlatMap[1]了，我们只能分配到其他SharedSlot中去。从TaskManager2中分配出一个SharedSlot，并从该SharedSlot中为FlatMap[2]分配出一个SimpleSlot。如上图的④和⑤。
1. 为Key->Sink[1]分配slot。目前两个SharedSlot都符合条件，从TaskManager1的SharedSlot中分配出一个SimpleSlot用来部署Key->Sink[1]。如上图中的⑥。
1. 为Key->Sink[2]分配slot。TaskManager1的SharedSlot中已经有同operator的Key->Sink[1]了，则只能选择另一个SharedSlot中分配出一个SimpleSlot用来部署Key->Sink[2]。
2. 最后Source、FlatMap[1]、Key->Sink[1]这些subtask都会部署到TaskManager1的唯一一个slot中，并启动对应的线程。FlatMap[2]、Key->Sink[2]这些subtask都会被部署到TaskManager2的唯一一个slot中，并启动对应的线程。从而实现了slot共享

#### 意义
- Flink 集群所需的taskslots数与job中最高的并行度一致。也就是说我们不需要再去计算一个程序总共会起多少个task了。
- .更容易获得更充分的资源利用。如果没有slot共享，那么非密集型操作source/flatmap就会占用同密集型操作 keyAggregation/sink 一样多的资源。如果有slot共享，将基线的2个并行度增加到6个，能充分利用slot资源，同时保证每个TaskManager能平均分配到重的subtasks，比如keyby/window/apply操作就会均分到申请的所有slot里，这样slot的负载就均衡了


## slot 和 parallelism
- slot是静态的概念，是指taskmanager**具有的并发执行能力**
> 假设我们将 taskmanager.numberOfTaskSlots 配置为3
那么每一个 taskmanager 中分配3个 TaskSlot, 3个 taskmanager 一共有9个TaskSlot
- parallelism是动态的概念，是指程序运行时实际使用的并发能力
> 假设我们把 parallelism.default 设置为1，那么9个 TaskSlot 只能用1个，有8个空闲
- 设置合适的parallelism能提高运算效率，太多了和太少了都不行

### parallelism 的设置
> 算子级别 > 执行环境级别 > 提交任务级别 > 系统配置级别。
- 启动Flink应用程序时
- env.setParallelism()
- Flink的配置文件里面去设置 parallelism.default property 
- Flink每个算子都可以设置并行度

#### 算子级别
```
DataSet<Tuple2<String, Integer>> counts =
      text.flatMap(new LineSplitter())
            .groupBy(0)
            .sum(1).setParallelism(1);
```
- 事实上，Flink 的每个算子都可以单独设置并行度。这也是我们最推荐的一种方式，可以针对每个算子进行任务的调优

#### 执行环境级别
- 在创建 Flink 的上下文时可以显示的调用env.setParallelism()方法，来设置当前执行环境的并行度，这个配置会对当前任务的所有算子、Source、Sink 生效。当然你还可以在算子级别设置并行度来覆盖这个设置。

```
final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(5);
```

#### 提交任务级别
```
./bin/flink run -p 10 WordCount.jar
```
#### 系统配置级别
- 在 flink-conf.yaml 中的一个配置：parallelism.default，该配置即是在系统层面设置所有执行环境的并行度配置。

