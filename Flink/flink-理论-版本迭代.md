[toc]
## flink1.9
### Sate Processor API
- Flink1.9 新添加的功能,其能够帮助用户直接访问Flink中存储的State，API能够帮助用户非常方便地读取、修改甚至重建整个State。这个功能的强大之处在于几个方面，第一个就是灵活地读取外部的数据，比如从一个数据库中读取自主地构建Savepoint，解决作业冷启动问题，这样就不用从N天前开始重跑整个数据
### 可以使用的场景
- 异步校验或者查看某个阶段的状态，一般而言，flink作业的最终结果都- 会持久化输出，但在面临问题的时候，如何确定哪一级出现问题，state - processor api也提供了一种可能，去检验state中的数据是否与预期的- 一致。
- 脏数据订正,比如有一条脏数据污染了State，就可以用State Processor - API对于状态进行修复和订正。
- 状态迁移，当用户修改了作业逻辑，还想要复用原来作业中大部分的Sta- te，或者想要升级这个State的结构就可以用这个API来完成相应的工作- 。
- 解决作业冷启动问题，这样就不用从N天前开始重跑整个数据

### 一些限制点
- window state暂时修改不了
- 每个有状态的算子都必须手动指定uid
- 无法通过读取savepoint 直接获取到metadata 信息(existing - operator ids)

## flink1.11
### 非对齐的 Checkpoint 机制
- 核心引擎部分引入了非对齐的 Checkpoint 机制。这一机制是对 Flink 容错机制的一个重要改进，它可以提高严重反压作业的 Checkpoint 速度。
- 当 Flink 发起一次 Checkpoint 时， Checkpoint Barrier 会从整个拓扑的 Source 出发一直流动到 Sink。对于超过一个输入的算子，来自各个输入的 Barrier 首先需要对齐，然后这个算子才能进行 state 的快照操作以及将 Barrier 发布给后续的算子。**一般情况下对齐可以在几毫秒内完成，但是当反压时，对齐可能成为一个瓶颈**
- Checkpoint Barrier 在有反压的输入通道中传播的速度非常慢（需要等待前面的数据处理完成），这将会阻塞对其它输入通道的数据处理并最终进一步反压上游的算子。
- Checkpoint Barrier 传播慢还会导致Checkpoint时间过长甚至超时，在最坏的情况下，这可能导致整个作业进度无法更新。
- 为了提高 Checkpoint 在反压情况下的性能，Flink 社区在 1.11.0 版本中初步实现了非对齐的 Checkpoint 机制（FLIP-76）。与对齐的 Checkpoint（图1）相比，这种方式下算子不需要等待来自各个输入通道的 Barrier 对齐，相反，这种方式允许 Barrier 越过前面的待处理的数据（即在输出和输入 Buffer 中的数据）并且直接触发 Checkpoint 的同步阶段。这一过程如图2所示。

![image-20210202200534244](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202200534244.png)
- 由于被越过的传播中的数据必须作为快照的一部分被持久化，**非对齐的 Checkpoint 机制会增加 Checkpoint 的大小**。
- 但是，好的方面是它可以**极大的减少Checkpoint需要的时间，因此即使在非稳定的环境中，用户也可以看到更多的作业进度**。这是由于非对齐的 Checkpoint 可以减少 Recovery 的负载。关于非对齐的 Checkpoint 更详细的信息以及未来的开发计划，可以分别参考相关文档和 FLINK-14551。


### 实现了一套新的 Source 接口
- 通过统一流和批作业 Source 的运行机制，提供常用的内部实现如事件时间处理，watermark 生成和空闲并发检测，这套新的 Source 接口可以极大的降低实现新的 Source 时的开发复杂度。
- AssignerWithPunctuatedWatermarks 与 AssignerWithPeriodicWatermarks，这两个接口与记录时间戳提取的关系也比较混乱，从而使 Flink 难以实现一些用户急需的功能，如支持空闲检测；此外，这还会导致代码重复且难以维护。
- 现有的 watermark 生成接口被统一为一个单独的接口，即 WatermarkGenerator，并且它和 TimestampAssigner 独立。
- 这一修改使用户可以更好的控制 watermark 的发送逻辑，并且简化实现支持watermark 生成和时间戳提取的 Source 的难度（可以参考新的 Source 接口）。基于这一接口，Flink 1.11 中还提供了许多内置的 Watermark 生成策略（例如 forBoundedOutOfOrderness, forMonotonousTimestamps），并且用户可以使用自己的实现。
- WatermarkStrategy.withIdleness()方法允许用户在配置的时间内（即超时时间内）没有记录到达时将一个流标记为空闲，从而进一步支持 Flink 正确处理多个并发之间的事件时间倾斜的问题，并且避免了空闲的并发延迟整个系统的事件时间。通过将 Kafka 连接器迁移至新的接口（FLINK-17669），用户可以受益于针对单个并发的空闲检测。


### CDC（Change Data Capture，变动数据捕获）
- Flink SQL 引入了对 CDC（Change Data Capture，变动数据捕获）的支持，它使 Flink 可以方便的通过像 Debezium 这类工具来翻译和消费数据库的变动日志。
- Table API 和 SQL 也扩展了文件系统连接器对更多用户场景和格式的支持，从而可以支持将流式数据从 Kafka 写入 Hive 等场景。

### PyFlink 优化了多个部分的性能
- 包括对向量化的用户自定义函数（Python UDF）的支持。这些改动使 Flink Python 接口可以与常用的 Python 库（如 Pandas 和 NumPy）进行互操作，从而使 Flink 更适合数据处理与机器学习的场景
  
