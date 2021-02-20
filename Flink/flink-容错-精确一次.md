[toc]
## 两阶段提交协议
- 两阶段提交指的是一种协议，经常用来实现分布式事务，可以简单理解为预提交+实际提交，一般分为协调器Coordinator简称和若干事务参与者Participant两种角色。
> 在分布式系统中，可以使用两阶段提交来实现事务性从而保证数据的一致性，协调者用于用于管理所有执行者的操作，执行者用于执行具体的提交操作

### 执行流程
- 首先协调者会送预提交(pre-commit)命令有的执行者    
- 执行者执行预提交操作然后发送一条反馈(ack)消息给协调者
- 待协调者收到所有执行者的成功反馈，则发送一条提交信息(commit)给执行者，如果有一个失败则执行rollback
- 执行者执行提交操作
```
如果在流程2中部分预提交失败，那么协调者就会收到一条失败的反馈，则会发送一条rollback消息给所有执行者，执行回滚操作，保证数据一致性；
但是如果在流程4中，出现部分提交成功部分提交失败，那么就会造成数据的不一致
```
## EOS
- 指的是每条输入消息只会影响最终结果一次，注意这里**是影响一次，而非处理一次**，Flink一直宣称自己支持EOS，实际上主要是对于Flink应用内部来说的，对于外部系统(端到端)则有比较强的限制
    - 外部系统写入支持幂等性
    - 外部系统支持以事务的方式写入 
> 在flink1.4.0之前，flink通过checkpoint保证了flink应用内部的exactly-once语义。现在加入了TwoPhaseCommitSinkFunctio可以保证端到端的exactly-once语义

> 一个分布式且含有多个并发执行sink的应用中，仅仅执行单次提交或回滚是不够的，因为所有组件都必须对这些提交或回滚达成共识，这样才能保证得到一个一致性的结果。Flink使用两阶段提交协议以及预提交(pre-commit)阶段来解决这个问题。

### kafka 的幂等
![image-20210218193635593](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218193635593.png)
- 为了实现Producer的幂等语义，Kafka引入了Producer ID（即PID）和Sequence Number。每个新的Producer在初始化的时候会被分配一个唯一的PID，该PID对用户完全透明而不会暴露给用户。
- Producer发送每条消息<Topic, Partition>对于Sequence Number会从0开始单调递增，broker端会为每个<PID, Topic, Partition>维护一个序号，每次commit一条消息此序号加一，对于接收的每条消息，如果其序号比Broker维护的序号（即最后一次Commit的消息的序号）大1以上，则Broker会接受它，否则将其丢弃：
    - 序号比Broker维护的序号大1以上，说明存在乱序。
    - 序号比Broker维护的序号小，说明此消息以及被保存，为重复数据。 

![image-20210218193652713](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218193652713.png)
- 幂等性机制仅解决了**单分区上的数据重复和乱序问题**，对于跨session和所有分区的重复和乱序问题不能得到解决。于是需要引入事务
- 为了支持事务，Kafka引入了Transacation Coordinator来协调整个事务的进行，并可将事务持久化到内部topic里，类似于offset和group的保存。
- 用户为应用提供一个全局的Transacation ID，应用重启后Transacation ID不会改变。为了保证新的Producer启动后，旧的具有相同Transaction ID的Producer即失效，每次Producer通过Transaction ID拿到PID的同时，还会获取一个单调递增的epoch。由于旧的Producer的epoch比新Producer的epoch小，Kafka可以很容易识别出该Producer是老的Producer并拒绝其请求。
    - 跨Session的数据幂等发送。当具有相同Transaction ID的新的Producer实例被创建且工作时，旧的Producer停止工作。跨Session的事务恢复。
    - 如果某个应用实例宕机，新的实例可以保证任何未完成的旧的事务要么Commit要么Abort，使得新实例从一个正常状态开始工作。

![image-20210218193705576](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218193705576.png)

### kafka 事务
- kafka事务属性是指一系列的生产者生产消息和消费者提交偏移量的操作在一个事务，或者说是是一个原子操作）
- consumer-transform-producer模式下，因为消费者提交偏移量出现问题，导致在重复消费消息时，生产者重复生产消息。
- 需要将这个模式下消费者提交偏移量操作和生成者一系列生成消息的操作封装成一个原子操作。
> 消费者提交偏移量导致重复消费消息的场景：消费者在消费消息完成提交偏移量o2之前挂掉了（假设它最近提交的偏移量是o1），此时执行再均衡时，其它消费者会重复消费消息(o1到o2之间的消息）

#### 需要原子性操作的场景
1. 只有Producer生产消息
2. 消费消息和生产消息并存，这个是事务场景中最常用的情况，就是我们常说的“consume-transform-produce ”模式
3. 只有consumer消费消息，
>前两种情况是事务引入的场景，最后一种情况没有使用价值（跟使用手动提交效果一样）。

#### api 适用
1. 需要消费者的自动模式设置为false,并且不能再手动的执行consumer#commitSync或者consumer#commitAsyc
2. 生产者配置transaction.id属性
3. 生产者不需要再配置enable.idempotence，因为如果配置了transaction.id，则此时enable.idempotence会被设置为true
4. 消费者需要配置Isolation.level。在consume-trnasform-produce模式下使用事务时，必须设置为READ_COMMITTED


# flink 精确一次的语义
- flink 提供 exactly-once 的状态（state）投递语义，这为有状态的（stateful）计算提供了准确性保证。其中比较容易令人混淆的一点是状态投递语义和更加常见的端到端（end to end）投递语义，而实现前者是实现后者的前置条件。
-  Flink 采用快照的方式来将整个作业的状态定期同步到外部存储，也就是将 State API 保存的信息以序列化的形式存储，作业恢复的时候只要读取外部存储即可将作业恢复到先前某个时间点的状态。由于从快照恢复同时会回滚数据流的处理进度，所以 State 是天然的 exactly-once 投递。
- 端到端的一致性则需要上下游的外部系统配合，因为 Flink 无法将它们的状态也保存到快照并独立地回滚它们，否则就不叫作外部系统了
- 通常来说 Flink 的上游是可以重复读取或者消费的 pull-based 持续化存储，所以要实现 source 端的 exactly-once 只需要回滚 source 的读取进度即可（e.g. Kafka 的 offset）。而 sink 端的 exactly-once 则比较复杂，因为 sink 是 push-based 的。所谓覆水难收，要撤回发出去的消息是并不是容易的事情，因为这要求下游根据消息作出的一系列反应都是可撤回的。



## 幂等
- 它意味着相同的操作执行一次和执行多次可以获得相同的结果，因此 at-least-once 自然等同于 exactly-once。如此一来，在从快照恢复的时候幂等 sink 便不需要对外部系统撤回已发消息，相当于回避了外部系统的状态回滚问题。
- 幂等性要求输出的结果数据具有唯一性，也就是要求具有唯一键或者联合唯一键，这类应用最常见的就是窗口聚合结果数据输出，并且结果带上窗口信息，由于窗口不会重复并且多次计算结果也不会改变（对于EventTime）, 只需要接收端能够保证唯一性即可，例如Redis/HBase 的Key, Mysql中Unique Key
- 然而幂等 sink 的适用场景依赖于业务逻辑，如果下游业务本来就无法保证幂等性，这时就需要应用事务性 sink。
- 在 Flink 1.4.0 之前，Exactly-Once 语义仅限于 Flink 应用程序内部，并没有扩展到 Flink 数据处理完后发送的大多数外部系统。Flink 应用程序与各种数据输出端进行交互，开发人员需要有能力自己维护组件的上下文来保证 Exactly-Once 语义,也就是说单独借助checkpoint 是不能保证端到端的一致性语义的，只能保证flink 内部的一致性语义

## 最终一致性
- 最终一致性实现方案相对来说是比较简单的，其依托Flink 的Checkpoint机制与内部状态存储来实现，Flink本身是支持内部Exactly-Once语义，那我们可以将所有的结果数据都保存在状态中，数据到输出端的过程仅仅是从状态中取数据的过程，当任务失败重启，Flink状态可恢复到最近一次成功checkpoint点，如果在该checkpoint点之后已经有数据输出，那么就可能会出现状态数据与输出端数据不一致的情况，但是随着任务的执行，通过**覆盖**的形式输出，最终保证状态与输出端的数据的一致性。
- 其实就是相当于将输出uodate 到外部
```
 env.addSource(consumer)
        .map((_,1))
        .keyBy(_._1)
        .sum(1)
        .print()

```
## flink 2-pc(事物)
- flink中两阶段提交是为了保证端到端的Exactly Once，主要依托checkpoint机制来实现

### checkpoint 流程
- jobMaster 会周期性的发送执行checkpoint命令(start checkpoint)；
- 当source端收到执行指令后会产生一条barrier消息插入到input消息队列中，当处理到barrier时会执行本地checkpoint, 并且会将barrier发送到下一个节点，当checkpoint完成之后会发送一条ack信息给jobMaster ；
- 当DAG图中所有节点都完成checkpoint之后，jobMaster会收到来自所有节点的ack信息，那么就表示一次完整的checkpoint的完成；
- JobMaster会给所有节点发送一条callback信息，表示通知checkpoint完成消息，这个过程是异步的，并非必须的，方便做一些其他的事情，例如kafka offset提交到kafka。
```
 对比flink 整个checkpoint机制调用流程可以发现与2PC非常相似，JobMaster相当于协调者，所有的处理节点相当于执行者，start-checkpoint消息相当于pre-commit消息，每个处理节点的checkpoint相当于pre-commit过程，checkpoint ack消息相当于执行者反馈信息，最后callback消息相当于commit消息，完成具体的提交动作
```

### TwoPhaseCommitSinkFunction
- 完整的实现两阶段提交协议可能有点复杂，这就是为什么 Flink 将它的通用逻辑提取到抽象类 TwoPhaseCommitSinkFunction 中的原因
- 用户只需要实现四个函数，就能为数据输出端实现 Exactly-Once 语义：

#### 实现原理
- flink 提供了CheckpointedFunction与CheckpointListener这样两个接口，CheckpointedFunction中有snapshotState方法，每次checkpoint触发执行方法，通常会将缓存数据放入状态中，可以理解为是一个hook，这个方法里面可以实现预提交，CheckpointListener中有notifyCheckpointComplete方法，checkpoint完成之后的通知方法，这里可以做一些额外的操作，例如FlinkKafakConsumerBase 使用这个来完成kafka offset的提交，在这个方法里面可以实现提交操作。
- 在2PC中提到如果对应流程2预提交失败，那么本次checkpoint就被取消不会执行，不会影响数据一致性，那么如果流程4提交失败了，在flink中可以怎么处理的呢？ 我们可以在预提交阶段(snapshotState)将事务的信息保存在state状态中，如果流程4失败，那么就可以从状态中恢复事务信息，并且在CheckpointedFunction的initializeState方法中完成事务的提交，该方法是初始化方法只会执行一次，从而保证数据一致性。
- Flink 是可以通过状态与checkpoint机制实现内部Exactly-Once 的语义，对于端到端的Exactly-Once语义，Flink 通过两阶段提交方式提供了对Kafka/HDFS输出支持，两阶段提交实现是结合checkpoint流程提供的hook来实现的，实现CheckpointedFunction与CheckpointListener接口
- 使用flink自带的实现要求继承TwoPhaseCommitSinkFunction类，并且实现beginTransaction、preCommit、commit、abort这几个方法，虽然说使用起来很方便，但是其有一个限制那就是所提供的事务hook(比喻Connection)能够被序列化，并且反序列化之后能够继续提交之前的事务，这个对于很多事务性的数据库是无法做到的，所以需要实现一套特有的事务提交
- commit有可能会存在部分成功与部分失败，所以才有了事务容错恢复，提交失败的重启继续提交，提交成功的重启再次提交是幂等的不会影响数据的结果，现在没有了这样一个可序列化的事务hook,另外需要提交的数据也做了状态容错。但是Flink 在checkpoint机制中提供了一个唯一的标识checkpointId, 它是用户可访问的、单调递增的、容错的，任务失败之后会从最近一次成功点继续递增，那么就可以使用checkpointId 来作为事务提交的句柄，

### Bucketing File Sink
- Bucketing File Sink 是 Flink 提供的一个 FileSystem Connector，用于将数据流写到固定大小的文件里。Bucketing File Sink 将文件分为三种状态，in-progress/pending/committed，分别表示正在写的文件、写完准备提交的文件和已经提交的文件。
- Bucketing File Sink 首先会打开一个临时文件并不断地将收到的数据写入（相当于事务的 beginTransaction 步骤），这时文件处于 in-progress。直到这个文件因为大小超过阈值或者一段时间内没有新数据写入，这时文件关闭并变为 pending 状态（相当于事务的 pre-commit 步骤）。由于 Flink checkpoint 是异步的，可能有多个并发的 checkpoint，Bucketing File Sink 会记录 pending 文件对应的 checkpoint epoch，当某个 epoch 的 checkpoint 完成后，Bucketing File Sink 会收到 callback 并将对应的文件改为 committed 状态。这是通过原子操作重命名来完成的，因此可以保证 pre-commit 的事务要么 commit 成功要么 commit 失败，不会出现其他中间状态。
- Commit 出现错误会导致作业自动重启，重启后 Bucketing File Sink 本身已被恢复为上次 checkpoint 时的状态，不过仍需要将文件系统的状态也恢复以保证一致性。从 checkpoint 恢复后对应的事务会再次重试 commit，它会将记录的 pending 文件改为 committed 状态，记录的 in-progress 文件 truncate 到 checkpoint 记录下来的 offset，而其余未被记录的 pending 文件和 in-progress 文件都将被删除。

### StreamingFileSink

## 例子

### kakfa 的端到端示例

![image-20210218193736164](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218193736164.png)

- 要使数据输出端提供 Exactly-Once 保证，它必须将所有数据通过一个事务提交给 Kafka。提交捆绑了两个 checkpoint 之间的所有要写入的数据。这可确保在发生故障时能回滚写入的数据。
- 但是在分布式系统中，通常会有多个并发运行的写入任务的，简单的提交或回滚是不够的，因为所有组件必须在提交或回滚时“一致”才能确保一致的结果。Flink 使用两阶段提交协议及预提交阶段来解决这个问题。

#### 预提交
![image-20210218193752610](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218193752610.png)
- 在 checkpoint 开始的时候，即两阶段提交协议的“预提交”阶段。当 checkpoint 开始时，Flink 的 JobManager 会将 checkpoint barrier（将数据流中的记录分为进入当前 checkpoint 与进入下一个 checkpoint ）注入数据流。
- brarrier 在 operator 之间传递。对于每一个 operator，它触发 operator 的状态快照写入到 state backend

##### 数据源
![image-20210218193805231](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218193805231.png)
- 数据源保存了消费 Kafka 的偏移量(offset)，之后将 checkpoint barrier 传递给下一个 operator。

##### 计算算子
- 当一个进程有它的内部状态的时候，除了在 checkpoint 之前需要将数据变更写入到 state backend ，不需要在预提交阶段执行任何其他操作
- Flink 负责在 checkpoint 成功的情况下正确提交这些写入，或者在出现故障时中止这些写入。

##### 外部状态
- 当进程具有『外部』状态时，需要作些额外的处理。外部状态通常以写入外部系统（如 Kafka）的形式出现。在这种情况下，为了提供 Exactly-Once 保证，外部系统必须支持事务，这样才能和两阶段提交协议集成。
- 因此数据输出端（ Data Sink ）有外部状态。在这种情况下，在预提交阶段，除了将其状态写入 state backend 之外，数据输出端还必须预先提交其外部事务。

![image-20210218193824880](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218193824880.png)

##### 结束
- 当 checkpoint barrier 在所有 operator 都传递了一遍，并且触发的 checkpoint 回调成功完成时，预提交阶段就结束了。
- 所有触发的状态快照都被视为该 checkpoint 的一部分。checkpoint 是整个应用程序状态的快照，包括预先提交的外部状态。
- 如果发生故障，我们可以回滚到上次成功完成快照的时间点。

#### 提交
- 下一步是通知所有 operator，checkpoint 已经成功了。
- 这是两阶段提交协议的提交阶段，JobManager 为应用程序中的每个 operator 发出 checkpoint 已完成的回调。
- 数据源和 windnow operator 没有外部状态，因此在提交阶段，这些 operator 不必执行任何操作。
- 但是，数据输出端（Data Sink）拥有外部状态，此时**应该提交外部事务**。

![image-20210218193842684](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218193842684.png)

#### 不同阶段fail over的recovery举措:
- 在pre-commit前fail over, 系统恢复到最近的checkponit
- 在pre-commit后,commit前failover,系统恢复到刚完成pre-commit时的状态(这个时候就要求要最终成功)

#### 总结
- 在预提交成功之后，提交的 commit **需要保证最终成功** - operator 和外部系统都需要保障这点。
- 如果 commit 失败（例如，由于间歇性网络问题），整个Flink应用程序将失败，应用程序将根据用户的**重启策略重新启动，还会尝试再提交**（可以设置一直提交）这个过程至关重要，因为**如果 commit 最终没有成功，将会导致数据丢失**（超过了重启策略的设置）——其实应该是程序再也不能正常运行(一直提交)
- Pre-commit失败，将恢复到最近一次CheckPoint位置;**一旦pre-commit完成，必须要确保commit也要成功**
- 需要在预提交阶段保存足够多的信息到checkpoint状态中，以便在重启后能正确的中止或提交事务。在这个例子中，这些信息是临时文件和目标目录的路径。
> 在预提交状态成功后，外部系统需要完美支持正式提交之前的操作。如果有提交失败发生，整个flink应用会进入失败状态并重启，重启后将会继续从上次状态来尝试进行提交操作。


## 扩展
### 三阶段提交协议
- 三阶段提交是为解决两阶段提交协议的缺点而设计的
-  2pc虽然在部分网络故障情况下存在强一致性被破坏的问题，但故障恢复以后也能保证最终一致性。
- 3pc引入超时时间，虽然解决了阻塞，提高了可用性，但一致性牺牲以后，带来的可能是比较严重的数据问题了。所以，如果不得不使用分布式事务，2pc反而比3pc更值得选择吧，如果选3pc还不如选择避免使用分布式事务

![image-20210218193909161](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218193909161.png)
- 三阶段提交协议引入了超时机制。在第一阶段和第二阶段中，引入了一个准备阶段。保证了在最后提交阶段之前各参与节点的状态是一致的。简单讲：就是除了引入超时机制之外，3PC把2PC的准备阶段再次一分为二，这样三阶段提交就有CanCommit、PreCommit、DoCommit三个阶段。

#### 第一阶段（CanCommit 阶段）
1. 事务询问：
	协调者向参与者发送CanCommit请求。询问是否可以执行事务提交操作。然后开始等待参与者的响应。
2. 响应反馈
	参与者接到CanCommit请求之后，正常情况下，
	如果其自身认为可以顺利执行事务，则返回Yes响应，并进入预备状态。
	否则反馈No

#### 第二阶段（PreCommit 阶段）
1. 发送预提交请求：
	协调者向参与者发送PreCommit请求，并进入Prepared阶段。
2. 事务预提交
	参与者接收到PreCommit请求后，会执行事务操作，并将undo和redo信息记录到事务日志中。
3. 响应反馈
	如果参与者成功的执行了事务操作，则返回ACK响应，同时开始等待最终指令。
> 假如有任何一个参与者向协调者发送了No响应，或者等待超时之后，协调者都没有接到参与者的响应，那么就执行事务的中断

```
1.发送中断请求：
	协调者向所有参与者发送abort请求。

2.中断事务
	参与者收到来自协调者的abort请求之后（或超时之后，仍未收到协调者的请求），执行事务的中断。

```

#### 第三阶段（doCommit 阶段）
1. 发送提交请求
	协调者接收到参与者发送的ACK响应，那么它将从预提交状态进入到提交状态。
	并向所有参与者发送doCommit请求。
2. 事务提交
	参与者接收到doCommit请求之后，执行正式的事务提交。
	并在完成事务提交之后释放所有事务资源。
3. 响应反馈
	事务提交完之后，向协调者发送ACK响应。
4. 完成事务
	协调者接收到所有参与者的ACK响应之后，完成事务。
> 协调者没有接收到参与者发送的ACK响应（可能是接受者发送的不是ACK响应，也可能响应超时），那么就会执行中断事务

```
1.发送中断请求
	协调者向所有参与者发送abort请求

2.事务回滚
	参与者接收到abort请求之后，利用其在阶段二记录的undo信息来执行事务的回滚操作，
	并在完成回滚之后释放所有的事务资源。

3.反馈结果
	参与者完成事务回滚之后，向协调者发送ACK消息

4.中断事务
	协调者接收到参与者反馈的ACK消息之后，执行事务的中断。

```
