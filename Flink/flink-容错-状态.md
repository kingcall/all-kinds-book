[toc]

## 状态计算
### 复杂事件处理
### 聚合计算
### 机器学习模型训练
### 使用历史数据进行计算
## state
- 状态本质上就是计算过程中的数据信息(包括中间结果和原始数据)，单条数据仅仅包含当前的信息，而不是包含所有的信息，要得到所需要的结果，还需要以前的计算结果(全部的历史数据)，那么这个过去的计算结果就是状态，所谓的状态指的是，在流处理过程中那些需要记住的数据，而这些数据既可以包括业务数据，也可以包括元数据。

- flink内置了状态管理，所以对flink应用程序来说，**状态的修改、查找都是本地操作，并且可以在不中断数据的情况下管理备份**。
- 所谓的状态指的是在进行流式计算过程中的信息。一般用作容错恢复和持久化，流式计算在本质上是增量计算，也就是说需要不断地查询过去的状态。状态在 Flink 中有十分重要的作用，例如为了确保 Exactly-once 语义需要将数据写到状态中；此外，状态的持久化存储也是集群出现 Fail-over 的情况下自动重启的前提条件。

- 当在 应用程序内部管理数据和状态时，减少了链接外部数据库的需求，进而实现非常高的计算性能
- 流式计算的数据往往是转瞬即逝， 当然，真实业务场景不可能说所有的数据都是进来之后就走掉，没有任何东西留下来，那么留下来的东西其实就是称之为 state
- 所有的State对象，仅仅用于与状态进行交互（更新、删除、清空等），而真正的状态值，有可能是存在内存、磁盘、或者其他分布式存储系统中。相当于我们只是持有了这个状态的句柄(state handle)
- Flink通过StateDescriptor来定义一个状态。这是一个抽象类，内部定义了状态名称、类型、序列化器等基础信息,与对应的状态对应，从StateDescriptor派生了ValueStateDescriptor, ListStateDescriptor等descriptor
- 由于状态需要从RuntimeContext中创建和获取，因此如果要使用状态，必须使用RichFunction。普通的Function是无状态的。
- **是否需要手动声明快照（snapshot）和恢复 (restore) 方法**：operator state 需要手动实现 snapshot 和 restore 方法；而 keyed state 则由 backend 自行实现，对用户透明。
- **数据大小**：一般而言，我们认为 operator state 的数据规模是比较小的；认为 keyed state 规模是相对比较大的。需要注意的是，这是一个经验判断，不是一个绝对的判断区分标准

- **存储对象是否 on heap**: 目前 operator state backend 仅有一种 on-heap 的实现；而 keyed state backend 有 on-heap 和 off-heap（RocksDB）的多种实现。

### 分配算子ID 
   - 如果您未手动指定ID，则会自动生成这些ID。只要这些ID不变，您就可以从保存点自动恢复。生成的ID取决于程序的结构，并且对程序更改很敏感。因此，强烈建议手动分配这些ID
   - 例子
   ````text
    env.
      // Stateful source (e.g. Kafka) with ID
      .addSource(new StatefulSource())
      .uid("source-id") // ID for the source operator
      .shuffle()
      // Stateful mapper with ID
      .map(new StatefulMapper())
      .uid("mapper-id") // ID for the mapper
      .print();
````

### state 的维护
- 在应用运行一段时间后，其状态就会变得十分珍贵，甚至无法重复计算
- 在修复bug、添加删除或调整功能、或者针对不同到来的数据来调整算子的并行度，所以说能将状态调整到一个版本的应用上或者重写分配到不同数量的算子上就显得十分重要

### state 扩容

#### 键值分区状态算子
- flink 会把所有键值状态分为键值组，然后以组为单位进行分配

#### 算子列表状态
- 算子列表状态的算子会将所有的状态统一收集起来，然后进行均匀分配；如果列表状态条目少于新的并行度，那么部分任务在启动的时候状态就为空

#### 算子联合列表状态
- 算子联合列表状态在扩容的时候会将状态列表的全部条目广播到全部任务上，有任务选择那些状态该保留，那些状态该丢弃。

#### 算子广播状态
- 算子广播状态 会将所有状态广播到新任务上。

### state 的划分
- Operator State 的实际应用场景不如 Keyed State 多，一般来说它会被用在 Source 或 Sink 等算子上，用来保存流入数据的偏移量或对输出数据做缓存，以保证 Flink 应用的 Exactly-Once 语义。

#### 基本类型划分
##### Keyed State
- 和Key有关的状态类型，它只能被基于KeyedStream之上的操作，方法所使用。我们可以从逻辑上理解这种状态是一个并行度操作实例和一种Key的对应， <parallel-operator-instance, key>。
- 非keyed 流上应用

```
java.lang.NullPointerException: Keyed state can only be used on a 'keyed stream', i.e., after a 'keyBy()' operation
```

##### Operator State（或者non-keyed state）
- Operator State 可以用在所有算子上，每个算子子任务或者说每个算子实例共享一个状态，流入这个算子子任务的数据可以访问和更新这个状态。每个算子子任务上的数据共享自己的状态。

- 它是和Key无关的一种状态类型。相应地我们从逻辑上去理解这个概念，它相当于一个并行度实例，对应一份状态数据。因为这里没有涉及Key的概念，所以在并行度（扩/缩容）发生变化的时候，这里会有状态数据的重分布的处理。

- Operator State是指在一个job中的一个task中的每一个operator对应着一个state，比如在一个job中，涉及到map，filter，sink等操作，那么在这些operator中，每一个可以对应着一个state（一个并行度），如果是多个并行度，那么每一个并行度都对应着一个state。对于Operator State主要有ListState可以进行使用

- 在每个算子中Managed Operator State都是以List形式存储，算子和算子之间的状态数据相互独立，List存储比较适合状态数据的重新分布，Flink目前支持对Managed OperatorState两种重分布的策略，分别是Even-split Redistribution和Union Redistribution。

###### 扩容与缩放

- **偶分裂再分配：**每个运算符返回一个状态元素列表。整个状态在逻辑上是所有列表的串联。在恢复/重新分发时，列表被平均分成与并行运算符一样多的子列表。每个运算符都获得一个子列表，该子列表可以为空，或包含一个或多个元素。例如，如果使用并行性1，则运算符的检查点状态包含元素element1和element2，当将并行性增加到2时，element1可能最终在运算符实例0分区中，而element2将转到运算符实例1分区。

- **联合重新分配：**每个运算符返回一个状态元素列表。整个状态在逻辑上是所有列表的串联。在恢复/重新分配时，每个运算符都会获得完整的状态元素列表。
> 如果要在还原时使用联合重新分发方案的列表状态，可以使用getUnoinListState(descriptor)方法获取。如果是getListState(descriptor)，它仅表示将使用基本偶分裂再分配方案

###### 使用 Operator State
- 如何使用Operator State呢？我们可以通过实现CheckpointedFunction接口来实现，或者实现ListCheckpointed<T extends Serializable>接口来实现，
- 它们之间主要的区别是：实现CheckpointedFunction接口，有两种形式的ListState API可以使用，分别是getListState以及getListUnionState，它们都会返回一个ListState，但是他们在重新分区的时候会有区别。
- 如果我们直接实现ListCheckpointed接口，那么就会规定使用ListState，不需要我们进行初始化，Flink内部帮我们解决。


#### 管理方式划分(组织形式划分)
##### Managed State
- 这类State的内部结构完全由Flink runtime内部来控制，包括如何将它们编码写入到checkpoint中等等。

##### Raw State
- 这类State就比较显得灵活一些，它们被保留在操作运行实例内部的数据结构中。从Flink系统角度来观察，在checkpoint时，它只知道的是这些状态数据是以连续字节的形式被写入checkpoint中。等待进行状态恢复时，又从字节数据反序列化为状态对象。


### Broadcast State
- Broadcast State具有Broadcast流的特殊属性，它是一种小数据状态广播向其它流的形式，从而避免大数据流量的传输。
- 在这里，其它流是对广播状态只有只读操作的允许，因为不同任务间没有跨任务的信息交流。一旦有运行实例对于广播状态数据进行更新了，就会造成状态不一致现象。

- Broadcast 广播变量：可以理解为是一个公共的共享变量，我们可以把一个dataset 或者不变的缓存对象（例如map list集合对象等）数据集广播出去，然后不同的任务在节点上都能够获取到，并在每个节点上只会存在一份，而不是在每个并发线程中存在。如果不使用broadcast，则在每个节点中的每个任务中都需要拷贝一份dataset数据集，比较浪费内存(也就是一个节点中可能会存在多份dataset数据)。

 

#### 应用场景
- 动态更新计算规则: 如事件流需要根据最新的规则进行计算，则可将规则作为广播状态广播到下游Task中。规则函数需要同时接收事件流和规则流，并将规则存为**算子状态**，然后将他们应用到事件流中全部的事件上。
- 实时增加额外字段: 如事件流需要实时增加用户的基础信息，则可将用户的基础信息作为广播状态广播到下游Task中。

#### 底层实现
- Broadcast State是Map类型，即K-V类型。

## Keyed State(键值分区状态)
- 就是基于KeyedStream上的状态。这个状态是跟特定的key绑定的，对KeyedStream流上的每一个key，可能都对应一个state。
- Keyed State 是经过分区后的流上状态，都有自己的状态，并且只有指定的 key 才能访问和更新自己对应的状态。

### 重分布
#### 对key进行hash取模重分配
- keyed State仅仅对于keyedStream是可用的，一般是通过keyBy()实现。相同的key，会通过hash方法，对这个operator的并行度取模操作，对应到相同的operator实例中
- 正是由于这个操作hash(key) mod parallelism(operator)，因此，在扩容时的状态重分配上，keyed state比operator state有一个明显的优势，keyed state很容易实现状态的重分配。
- 然而，这种简单的重分配方案，在扩容后也存在一个问题：hash(key) mod parallelism(operator)这个操作，很容易导致新分配的subtask，其处理的state，并不是之前本地操作的状态。这样导致新的subtask在恢复状态时，效率比较低。

#### 对key进行标记并跟踪location
- 这个方法就是在检查点时，对于每个key监理一个索引号来跟踪本operator实例生成的状态中有哪些key，这样在扩容后恢复时，就可以有选择性的读取本地实例生成的key的状态。

#### 不足
- key到index的映射，可能会增长的非常大
- 这种方法会产生巨大的随机IO操作

#### Flink中的key-groups组
- Flink对于keyed state的扩容后状态重分配的解决办法介于两者之间，其引入了key-group的概念。key-group是状态分配的原子单位。
- 首先，key-group的数量在job启动前必须是确定的且运行中不能改变。由于key-group是state分配的原子单位，而每个operator并行实例至少包含一个key-group，因此operator的最大并行度不能超过设定的key-group的个数
- 总而言之，key-group在扩容时的灵活性与恢复时的负载之间提供了一种解决办法。每个key-group是以key的范围来组织，这就使得我们在恢复时不仅可以顺序读取key-group，而且可以跨多个key-group读取。


### ValueState
- 即类型为T的单值状态。这个状态与对应的key绑定，是最简单的状态了。它可以通过update方法更新状态值，通过value()方法获取状态值。

### ListState
- 即key上的状态值为一个列表。可以通过add方法往列表中附加值；也可以通过get()方法返回一个Iterable<T>来遍历状态值。

###  ReducingState
- 这种状态通过用户传入的reduceFunction，每次调用add方法添加值的时候，会调用reduceFunction，最后合并到一个单一的状态值。

### FoldingState
- 跟ReducingState有点类似，不过它的状态值类型可以与add方法中传入的元素类型不同（这种状态将会在Flink未来版本中被删除）。

### MapState
- 即状态值为一个map。用户通过put或putAll方法添加元素。

### AggregateState

## Operator State(算子状态)
- Operator State跟一个特定operator的一个并发实例绑定，整个operator只对应一个state。相比较而言，在一个operator上，可能会有很多个key，从而对应多个keyed state。
- 同一算子的并行任务实例在处理任何事件都可以访问相同的状态
- 不同于键值分区状态那样直接在状态后端进行注册然后交互，而是需要将算子状态实现为成员变量并通过接口提供的回调函数与状态后端进行交互
- operator state可以在任意stream上适用，但是只支持有限的数据结构， Operator States的数据结构不像Keyed States丰富，现在只支持List。

```
Flink中的Kafka Connector，就使用了operator state。它会在每个connector实例中，保存该实例中消费topic的所有(partition, offset)映射。
```
###  Operator States多种扩展方式
![image](https://note.youdao.com/yws/res/33885/7E686B2747EF4907AA6C7408E5C0270E)
#### 列表状态
- ListState在并发度在改变的时候，会将并发上的每个List都取出，然后把这些List合并到一个新的List,然后根据元素的个数在**均匀分配给新的Task**

#### 联合列表状态
- 相比于ListState更加灵活，把划分的方式交给用户去做，当改变并发的时候，会将原来的List拼接起来。然后不做划分，直接交给用户

#### 广播状态
- 如大表和小表做Join时，小表可以直接广播给大表的分区，在每个并发上的数据都是完全一致的。做的更新也相同，当改变并发的时候，把这些数据COPY到新的Task即可

### 两大接口

##### CheckpointedFunction
- 要在自定义的函数或算子中使用状态，可以实现 CheckpointedFunction 接口，这是一个比较通用的接口，既可以管理 Operator State，也可以管理 Keyed State，灵活性比较强。
- CheckpointedFunction 是用来指定有状态函数的最底层接口，它提供了用于注册和维护键值分区状态算子的钩子函数
- 唯一支持算子联合列表状态(恢复时需要被完整的复制到任务实例上)的接口

```
void snapshotState(FunctionSnapshotContext context) throws Exception;
void initializeState(FunctionInitializationContext context) throws Exception;
```

##### ListCheckpointed
-  ListCheckpointed接口和CheckpointedFunction接口相比再灵活性上相对较弱一点，只能支持List类型的状态，并且在数据恢复时仅支持even-redistribution策略。

```
List<T> snapshotState(long checkpointId,long timestamp) throws Exception;
void restoreState(List<T> state) throws Exception;
```

```
stateful function可以通过CheckpointedFunction接口或者ListCheckpointed接口来使用managed operator state；对于manageed operator state，目前仅仅支持list-style的形式，即要求state是serializable objects的List结构，方便在rescale的时候进行redistributed；关于redistribution schemes的模式目前有两种，分别是Even-split redistribution(在restore/redistribution的时候每个operator仅仅得到整个state的sublist)及Union redistribution(在restore/redistribution的时候每个operator得到整个state的完整list)
ListCheckpointed是CheckpointedFunction的限制版，它只能支持Even-split redistribution模式的list-style state
ListCheckpointed定义了两个方法，分别是snapshotState方法及restoreState方法；snapshotState方法在master触发checkpoint的时候被调用，用户需要返回当前的状态，而restoreState方法会在failure recovery的时候被调用，传递的参数为List类型的state，方法里头可以将state恢复到本地
```

### 多重继承
```
假如一个类比如: FlinkKafkaConsumerBase 继承了 RichParallelSourceFunction 和CheckpointedFunction，并且重写了RichParallelSourceFunction 的open方法和CheckpointedFunction initializeState方法，那么是先执行open呢还是initializeState呢
```
- 先调用 initializeState
社区不建议使用这个类了，已经 deprecated 了，如果你用 ListCheckpointed 的话，注释里写的比较清楚

### 原始状态(Raw)
### Flink托管状态(Managed State)
- Keyed State和Operator State，可以以两种形式存在：原始状态和托管状态。
- 托管状态是由Flink框架管理的状态，如ValueState, ListState, MapState等。


## 状态操作
### 操作函数
#### ProcessWindowFunction
#### WindowFunction
#### RichFunction
 - RichFlatMapFunction 
#### KeyedProcessFunction
#### ProcessFunction

### StateDescriptor   
   通过创建StateDescriptor，然后获取相应的状态进行操作
   - state 的名称
   - state 数据类型
   - state 自定义函数
#### 常见的 StateDescriptor

## 状态清理
### 控制状态的大小
- 控制状态的大小。 状态生存时间特性的主要使用场景，就是能够有效地管理不断增长的状态大小。
- 通常情况下，数据只需要暂时保存，例如用户处在一次网络连接会话中。当用户访问事件结束时，我们实际上就没有必要保存该用户的状态，来减少无谓的状态存储空间占用。
- Flink 1.8.0引入的基于生存时间的后台状态清理机制，使得我们能够自动地对无用数据进行清理。
- 此前，应用程序开发人员必须采取额外的操作并显式地删除无用状态以释放存储空间。这种手动清理过程不仅容易出错，而且效率低下。以我们上述用户登录的案例为例，因为这些不活跃用户的相关信息会被自动过期清理掉，我们就不再需要额外存储上次登录的时间戳。

### 使用状态清理
- 在Flink的DataStream API中，应用程序状态是由状态描述符（state descriptor）来定义的。状态生存时间是通过将StateTtlConfiguration对象传递给状态描述符来配置的。



#### 什么时候重置生存时间
- 默认情况下，当状态被修改时，生存时间就会被更新。我们也可以在读操作访问状态时更新相关项的生存时间，但这样要花费额外的写操作来更新时间戳。
#### 已经过期的数据是否可以访问
状态生存时间机制使用的是惰性策略来清除过期状态。这可能导致应用程序会尝试读取过期但尚未删除的状态。用户可以配置对这样的读取请求是否返回过期状态。无论哪种情况，过期状态都会在之后立即被删除。虽然返回已经过期的状态有利于数据可用性，但不返回过期状态更符合相关数据保护法规的要求。

#### 哪种时间语义被用于定义生存时间
- 在Apache Flink 1.8.0中，用户只能根据处理时间（Processing Time）定义状态生存时间。

### 堆内存状态后端的增量清理
- 此方法只适用于堆内存状态后端（FsStateBackend和MemoryStateBackend）
- 其基本思路是在存储后端的所有状态条目上维护一个全局的惰性迭代器。某些事件（例如状态访问）会触发增量清理，而每次触发增量清理时，迭代器都会向前遍历删除已遍历的过期数据。
```
StateTtlConfig ttlConfig = StateTtlConfig
    .newBuilder(Time.days(7))
    // check 10 keys for every state access
    .cleanupIncrementally(10, false)
    .build();
```
- 如果启用该功能，则每次状态访问都会触发清除。而每次清理时，都会检查一定数量的状态条目是否过期。其中有两个调整参数。第一个定义了每次清理时要检查的状态条目数。第二个参数是一个标志位，用于表示是否在每条记录处理（record processed）之后（而不仅仅是访问状态，state accessed），都还额外触发清除逻辑。

#### 存在的问题
- 首先是增量清理所花费的时间会增加记录处理的延迟。其次，如果没有状态被访问（state accessed）或者没有记录被处理（record processed），过期的状态也将不会被删除。


### RocksDB状态后端利用后台压缩来清理过期状态
- 如果使用RocksDB状态后端，则可以启用另一种清理策略，该策略基于Flink定制的RocksDB压缩过滤器（compaction filter）。RocksDB会定期运行异步的压缩流程以合并数据并减少相关存储的数据量，该定制的压缩过滤器使用生存时间检查状态条目的过期时间戳，并丢弃所有过期值。
- 需要设置以下配置选项
```
state.backend.rocksdb.ttl.compaction.filter.enabled
```
```
StateTtlConfig ttlConfig = StateTtlConfig
    .newBuilder(Time.days(7))
    .cleanupInRocksdbCompactFilter()
    .build();
```
### 使用定时器进行状态清理
- 另一种手动清除状态的方法是基于Flink的计时器，这也是社区评估的一个想法。
- 使用这种方法，将为每个状态访问注册一个清除计时器。这种方法的清理更加精准，因为状态一旦过期就会被立刻删除。但是由于计时器会与原始状态一起存储会消耗空间，开销也更大一些。

## 合理使用检查点和savepoint
   - 监视检查点行为的最简单方法是通过UI的检查点部分。

 



# 问题
## 保存点的文件怎么这么小？
- 保存点只是个手动触发的指针，里边只存放元数据。真正的状态数据，即快照，是放在检查点的路径中。
## 程序中添加的新的operator会增么样？
- 新增的operator的状态将被初始化。保存点会保存所有有状态的operator的state，而不会保存无状态的operator信息。新增加的operator在启动时，就像个之前是无状态的operator一样


