[toc]
## checkpoint(检查点)
- 是应用程序的**定时、异步、一致性的快照操作**(应用程序的当前状态,输入流的位置)，轻量级容错机制(全局异步，局部同步)
- 它使用了State状态机制以及Checkpoint策略提供了强大的容错机制，不过我们需要注意区分它们，**State状态是指一个FlinkJob中的task中的每一个operator的状态，而Checkpoint是指在某的时刻下，对整个job一个全局的快照**，当我们遇到故障或者重启的时候可以从备份中进行恢复。
- 所谓checkpoint，就是在某一时刻，将所有task的状态做一个快照(snapshot)，然后存储到memory/file system/rocksdb等
- 检查点通过允许恢复状态和相应的流位置使Flink中的状态容错，从而**为应用程序提供与无故障执行相同的语义**。
- checkpoint机制不断绘制流应用的快照，流应用的状态快照被保存在配置的位置（如：JobManager的内存里，或者HDFS上）。
- **Flink分布式快照机制的核心是barriers，这些barriers周期性插入到数据流中，并作为数据流的一部分随之流动**。 
- 每个需要checkpoint的应用，它在启动的时候，Flink的JobManager就会为它创建一个checkpointCoordinator。checkpointCoordinator它全权负责本应用的快照的制作，用户可以通过checkpointCoordinator中的setCheckpointInterval接口设置checkpoint的周期。

### 为什么要开启checkpoint 
- 实时任务不同于批处理任务，除非用户主动停止，一般会一直运行，运行的过程中可能存在机器故障、网络问题、外界存储问题等等，要想实时任务一直能够稳定运行，实时任务要有自动容错恢复的功能。
- 实时任务开启 Checkpoint功能，也能够减少容错恢复的时间。因为每次都是从最新的Chekpoint点位开始状态恢复，而不是从程序启动的状态开始恢复(如果没有checkpoint的话，state 可能丢失)

### barriers
- barrier 其实是分布式一致性快照算法的产物
- 其实主要作用就是将数据分开，形成各个算子的一致性快照
- barrier 发送的方式是**直接广播**

#### 单流 barriers
- 屏障作为数据流的一部分随着记录被注入到数据流中。屏障永远不会赶超通常的流记录，它会**严格遵循顺序**。
- 屏障将数据流中的记录隔离成一系列的记录集合，并将一些集合中的数据加入到当前的快照中，而另一些数据加入到下一个快照中。每一个屏障携带着快照的ID，快照记录着ID并且将其放在快照数据的前面。屏障不会中断流处理，因此非常轻量级。

#### 并行流 barriers
- 不止一个输入流的时的operator，需要在快照屏障上对齐(align)输入流，才会发射出去。
- 可以看到1,2,3会一直放在Inputbuffer，直到另一个输入流的快照到达Operator。

#### barrier 和一致性语义
- barrier将数据流分为前后两个checkpoint(chk n,chk n+1)的概念，**如果不等待那么就会导致chk n的阶段处理了chk n+1阶段的数据**（一个算子会有多个实例中的一个实例会接受多个上游的barrier）但是在source端所记录的消费偏移量又一致****，如果chk n成功之后，后续的任务处理失败，任务重启会消费chk n+1阶段数据，就会到致数据重复消息


#### ABS(Asynchronus Barrier Snapshot 异步屏障)
- 检查点：时间点，在失败恢复的时候，从redo log 中回滚某个时间点之后未写入的磁盘的事物，这个时间点就是检查点
- 快照：copy on write(读密集型任务)，redirect on write 写密集型任务

#### barrier
- Flink 实现了一个轻量级的分布式快照机制，其核心点在于 Barrier。 Coordinator在需要触发检查点的时候要求数据源注入向数据流中注入 barrier，barrier和正常的数据流中的消息一起向前流动，相当于将数据流中的消息切分到了不同的检查点中。当一个 operator 从它所有的 input channel 中都收到了 barrier，则会触发当前 operator 的快照操作，并向其下游 channel 中发射 barrier。当所有的 sink 都反馈收到了 barrier 后，则当前检查点创建完毕。

#### 对齐操作
- 一个关键的问题在于，一些 operator 拥有多个 input channel，它往往不会同时从这些 channel 中接收到 barrier。如果 Operator 继续处理 barrier 先到达的 channel 中的消息，那么在所有 channel 的 barrier 都到达时，operator 就会处于一种混杂的状态。在这种情况下，Flink 采用对齐操作来保证 Exactly Once 特性。Operator 会阻塞 barrier 先到达的 channel，通常是将其流入的消息放入缓冲区中，待收到所有 input channel 的 barrier 后，进行快照操作，释放被阻塞的 channel，并向下游发射 barrier。 stream_aligning
- 对齐操作会对流处理造成延时，但通常不会特别明显。如果应用对一致性要求比较宽泛的话，那么也可以选择跳过对齐操作。这意味着快照中会包含一些属于下一个检查点的数据，这样就不能保证 Exactly Once 特性，而只能降级为 *At Least Once*。
- 在有多个输入 Channel 的情况下，为了数据准确性，算子会等待所有流的 Barrier 都到达之后才会开始本地的快照，这种机制被称为 Barrier 对齐。在对齐的过程中，算子只会继续处理的来自未出现 Barrier Channel 的数据，而其余 Channel 的数据会被写入输入队列，直至在队列满后被阻塞。当所有 Barrier 到达后，算子进行本地快照，输出 Barrier 到下游并恢复正常处理。


### 检查点配置与恢复
#### 配置
- 默认情况下，检查点不会保存，**仅用于从失败中恢复作业。取消程序时会删除它们**。
可以配置要保存的定期检查点。根据配置，当作业失败或取消时，不会自动清除这些保存的检查点。这样，如果您的工作失败，您将有一个检查点可以从中恢复
- 检查点由元数据文件和一些其他数据文件组成，具体取决于状态后台。
- 元数据文件和数据文件存储在state.checkpoints.dir配置文件中配置的目录中，也可以为代码中的每个作业指定。
- 配置
    - state.checkpoints.dir: hdfs:///checkpoints/
    - env.setStateBackend(new RocksDBStateBackend("hdfs:///checkpoints-data/");

##### 配置文件配置
- state.backend用于指定checkpoint state存储的backend，默认为none
- state.backend.async用于指定backend是否使用异步snapshot(默认为true)，有些不支持async或者只支持async的state backend可能会忽略这个参数
- state.backend.fs.memory-threshold，默认为1024，用于指定存储于files的state大小阈值，如果小于该值则会存储在root checkpoint metadata file
- state.backend.incremental，默认为false，用于指定是否采用增量checkpoint，有些不支持增量checkpoint的backend会忽略该配置
- state.backend.local-recovery，默认为false
- state.checkpoints.dir，默认为none，用于指定checkpoint的data files和meta data存储的目录，该目录必须对所有参与的TaskManagers及JobManagers可见
- state.checkpoints.num-retained，默认为1，用于指定保留的已完成的checkpoints个数
- state.savepoints.dir，默认为none，用于指定savepoints的默认目录
- taskmanager.state.local.root-dirs，默认为none

##### 代码配置

#### 恢复
- 计算图的拓扑结构不变——引擎从持久化的后端中华读入一个可用的快照，然后重新初始化所有物理计算任务
- 计算图的拓扑结构改变——JobManager **根据快照重新编排计算任务** 

### checkpoint 流程

![image-20210202094801078](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202094801078.png)

#### 流程详解
![image-20210202094815846](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202094815846.png)
- Checkpoint机制的第一步，就是CheckpointCoordinator周期性的向该流应用，所有的source算子发送barrier。
- 第二步，Source算子接收到一个barrier后，便暂停处理数据，将当前的状态制作成快照，并保存到指定的持久化存储中，最后它再向CheckpointCoordinator报告自己的快照制作情况。同时向自身下游所有算子广播该barrier。然后恢复该算子的数据处理工作。
- 下游的算子接收到barrier后，也会暂停自的数据处理过程，同2过程。
最后CheckpointCoordinator会确认它接收到的报告，如果收到本周期的所有算子的快照就认为快照制作成功，否则失败。

- JM trigger checkpoint
- Source 收到 trigger checkpoint 的 PRC，自己开始做 snapshot，并往下游发送 barrier
- 下游接收 barrier（根据 Checkpoint 的语义，决定是否在进行 CheckpointBarrier 对齐时）开始做 Checkpoint,
    - Task 开始同步阶段 snapshot
    - Task 开始异步阶段 snapshot
- 只有当所有 Sink 算子完成 Checkpoint 且发送确认消息后，该次 Checkpoint 才算完成。

#### CheckpointCoordinator
- Flink的checkpoint，是由CheckpointCoordinator来协调的，它位于JobMaster中。但是其实在ExecutionGraph中已经创建了，见ExecutionGraph.enableSnapshotCheckpointing方法。
- 当Job状态切换到RUNNING时，CheckpointCoordinatorDeActivator（从JobStatusListener派生）会触发回调coordinator.startCheckpointScheduler();，根据配置的checkpoint interval来定期触发checkpoint。

#### 例子
![image-20210202094840764](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202094840764.png)

### 对比savepoint
- 使用状态后台特定（低级）数据格式，可以是增量式的。不支持Flink特定函数，如重新缩放。 

### checkpoint 异常
#### checkpoint 异常的原因
- 用户代码逻辑没有对于异常处理，让其直接在运行中抛出。比如解析 Json 异常，没有捕获，导致 Checkpoint失败，或者调用 Dubbo 超时异常等等。
- 依赖外部存储系统，在进行数据交互时，出错，异常没有处理。比如输出数据到 Kafka、Redis、HBase等，客户端抛出了超时异常，没有进行捕获，Flink任务容错机制会再次重启(可能导致flink频繁重启)
- 内存不足，频繁GC，超出了 GC 负载的限制。比如 OOM 异常
- 网络问题、机器不可用问题等等

```
从目前的具体实践情况来看，Flink Checkpoint 异常觉大多数还是用户代码逻辑的问题，对于程序异常没有正确的处理导致。所以在编写 Flink 实时任务时，一定要注意处理程序可能出现的各种异常。这样，也会让实时任务的逻辑更加的健壮。
```

#### checkpoint 超时
- jobmanager定时trigger checkpoint，给source处发送trigger信号，同时会启动一个异步线程，在checkpoint timeout时长之后停止本轮checkpoint，cancel动作执行之后本轮的checkpoint就为超时，如果在超时之前收到了最后一个sink算子的ack信号，那么checkpoint就是成功的。

##### Barrier对齐
- StreamTask收集到相应的inputChannel的barrier，收集齐之后就将barrier下发，并开始自己task的checkpoint逻辑，如果上下游是rescale或者forward的形式，下游只需要等待1个并发的barrier，因为是point-to-point的形式，如果是hash或者rebalance，下游的每一个task开始checkpoint的前提就是要**收集齐上游所有并发的barrier**。

##### 异步状态遍历和写hdfs
- 状态很大的时候就会出现这个现象,现象比较明显

##### 全量Checkpoint
- 可以使用rocksdb，做增量的checkpoint,否则其他的state backend是做全量的，导致每次checkpoint时，数据量会逐步增加，也就变得超时了

##### 外部交互
- 是一种比较常见的现象，例如和数据库或者接口进行交互的时候会由于网络问题或者连接问题导致超时

##### 反压机制
##### 数据倾斜
##### checkpoint 设置不合理

### 异步快照
- 前述的检查点创建流程中，在 operator 进行快照操作时，不再处理数据流中的消息。这种同步的方式会在每一次进行快照操作的时候引入延时。实际上，Flink 也支持采用异步的方式创建快照，这就要求 operator 在触发快照操作的时候创建一个**不受后续操作影响的状态对象**，通常选用 copy-on-write 的数据结构。Flink 中基于 RocketDB 的状态存储后端就可以支持异步操作。
- 将 checkpoint 数据写入持久存储是异步发生的，这意味着 Flink 应用程序在 checkpoint 过程中可以继续处理数据。

## Distributed Snapshot
- 所谓分布式快照，实际上就是特定时间点记录下来的分布式系统的全局状态（global state）。分布式快照的主要用途有**故障恢复（即检查点）、死锁检测、垃圾收集**等

### Global Snapshot
- 在 Chandy-Lamport 算法中，为了定义分布式系统的全局状态，我们先将分布式系统简化成有限个进程和进程之间的 channel 组成，也就是一个有向图：节点是进程，边是 channel。因为是分布式系统，也就是说，这些进程是运行在不同的物理机器上的。那么一个分布式系统的全局状态就是有进程的状态和 channel 中的 message 组成，这个也是分布式快照算法需要记录的。
- 因为是有向图，所以每个进程对应着两类 channel: input channel, output channel。同时假设 Channel 是一个容量无限大的 FIFO 队列，收到的 message 都是有序且无重复的。**Chandy-Lamport 分布式快照算法通过记录每个进程的 local state 和它的 input channel 中有序的message，我们可以认为这是一个局部快照**。那么全局快照就可以通过将所有的进程的局部快照合并起来得到。

### Chandy-Lamport 
- Flink 的分布式快照是根据Chandy-Lamport算法量身定做的。简单来说就是**持续创建分布式数据流及其状态的一致快照**。
- 核心思想是在 input source 端插入 barrier，控制 barrier 的同步来实现 snapshot 的备份和 exactly-once 语义。

#### 算法详解
- Chandy-Lamport 算法将分布式系统抽象成 DAG（暂时不考虑有闭环的图），节点表示进程，边表示两个进程间通信的管道。
- 分布式快照的目的是**记录下整个系统的状态**，即可以分为节点的状态（进程的状态）和边的状态（信道的状态，即传输中的数据）。
- 因为系统状态是由输入的消息序列驱动变化的，我们可以将输入的消息序列分为多个较短的子序列，图的每个节点或边先后处理完某个子序列后，都会进入同一个稳定的全局统状态。利用这个特性，系统的进程和信道在子序列的**边界点分别进行本地快照，即使各部分的快照时间点不同，最终也可以组合成一个有意义的全局快照**。

![image-20210202094904804](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202094904804.png)
> 其实可以看出source 和sink 可能同时在做的checkpoint 都不是同一个heckpoint 的。

- 从实现上看，Flink 通过在 DAG 数据源定时向数据流注入名为 Barrier 的特殊元素，将连续的数据流切分为多个有限序列，对应多个 Checkpoint 周期。每当接收到 Barrier，算子进行本地的 Checkpoint 快照，**并在完成后异步上传本地快照**，同时将 Barrier 以广播方式发送至下游。当某个 Checkpoint 的所有 Barrier 到达 DAG 末端且所有算子完成快照，则标志着全局快照的成功。
- 比起其他分布式快照，该算法的优势在于辅以 Copy-On-Write 技术的情况下不需要 “Stop The World” 影响应用吞吐量，同时基本不用持久化处理中的数据，只用保存进程的状态信息，大大减小了快照的大小。

#### Initiating a snapshot:
- 也就是开始创建 snapshot，可以由系统中的任意一个进程发起
- 进程 Pi 发起: 记录自己的进程状态，同时生产一个标识信息 marker，marker 和进程通信的 message 不同,将 marker 信息通过 ouput channel 发送给系统里面的其他进程
开始记录所有 input channel 接收到的 message

#### Propagating a snapshot
- 系统中其他进程开始逐个创建 snapshot 的过程

#### Terminating a snapshot
- 算法结束条件
- 所有的进程都收到 marker 信息并且记录下自己的状态和 channel 的状态（包含的 message）


## savepoint(保存点)
- 保存点是**外部存储**的自包含检查点，可用于停止和恢复或更新Flink程序
- **使用Flink的检查点机制来创建流程序状态的（非增量）SNAPSHOT**，并将检查点数据和元数据写入外部文件系统
- 是借助底层的检查点实现的一层应用机制
- 主要作用
    - 平滑升级
    - 引擎升级
    - A/B 测试
- 从savepoint 启动任务的日志中会有 start job XXXX from savepoint from XXXX(路径)

### 配置保存点
- state.savepoints.dir: hdfs:///flink/savepoints 默认的保存点路径
- flink savepoint :jobId [:targetDirectory] 指定保存点的路径    

## 对比savepoint
![image-20210202094917844](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202094917844.png)
- 相比 Savepoint 而言，Checkpoint 更加轻量级，但有些场景 Checkpoint 并不能完全满足我们的需求。
- 所以在使用过程中，如果我们的需求能使用 Checkpoint 来解决优先使用 Checkpoint。当 Flink 任务中的一些依赖组件需要升级重启时，例如 hdfs、Kafka、yarn 升级或者 Flink 任务的 Sink 端对应的 MySQL、Redis 由于某些原因需要重启时，Flink 任务在这段时间也需要重启。
- 但是由于 Flink 任务的代码并没有修改，所以 Flink 任务启动时可以从 Checkpoint 处恢复任务。
- 如果 Checkpoint 目录的清除策略配置为 DELETE_ON_CANCELLATION，那么在取消任务时 Checkpoint 的状态信息会被清理掉，我们就无法通过 Checkpoint 来恢复任务了。为了从 Checkpoint 中轻量级地进行任务恢复，我们需要将该参数配置为 RETAIN_ON_CANCELLATION。
> 于状态超过 100G 的 Flink 任务 ，笔者在生产环境验证过：每次从 Savepoint 恢复任务时需要耗时 10分钟以上，而 Checkpoint 可以在 2分钟以内恢复完成。充分说明了 Checkpoint 相比 SavePoint 而言，确实是轻量级的，所以 Checkpoint 能满足的业务场景强烈建议使用 Checkpoint 恢复任务，而不是使用 SavePoint。


## Checkpoint 与反压的耦合
- 目前的 Checkpoint 算法在大多数情况下运行良好，然而当作业出现反压时，阻塞式的 Barrier 对齐反而会加剧作业的反压，甚至导致作业的不稳定。
- 首先， Chandy-Lamport 分布式快照的结束依赖于 Marker 的流动，而反压则会限制 Marker 的流动，导致快照的完成时间变长甚至超时
- 无论是哪种情况，都会导致 Checkpoint 的时间点落后于实际数据流较多。这时作业的计算进度是没有被持久化的，处于一个比较脆弱的状态，如果作业出于异常被动重启或者被用户主动重启，作业会回滚丢失一定的进度(比较大的进度)。
- 如果 Checkpoint 连续超时且没有很好的监控，回滚丢失的进度可能高达一天以上，对于实时业务这通常是不可接受的。更糟糕的是，回滚后的作业落后的 Lag 更大，**通常带来更大的反压，形成一个恶性循环**。
- 一直以来 Flink 的 Aligned Checkpoint 通过 Barrier 对齐，将本地快照延迟至所有 Barrier 到达，因而这个条件是永真的，从而巧妙地避免了对算子输入队列的状态进行快照，但代价是比较不可控的 Checkpoint 时长和吞吐量的降低。**实际上这和 Chandy-Lamport 算法是有一定出入的**。

### 两者的差异主要可以总结为两点:
- 快照的触发是在接收到第一个 Barrier 时还是在接收到最后一个 Barrier 时。
- 是否需要阻塞已经接收到 Barrier 的 Channel 的计算。