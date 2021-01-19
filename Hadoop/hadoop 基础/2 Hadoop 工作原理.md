# Hadoop 工作原理

在大数据领域，Hadoop 已经成为一种流行的解决方案。Hadoop 的设计考虑到了很多方面，比如故障容错性，海量数据处理，数据本地化，跨异构硬件和软件平台的可移植性等等。本节课程详细介绍了 Hadoop 的3个重要的组件。
![Hadoop架构图](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/hadoop-architecture_20191001225319369978-20210111222739825.png)

## Hadoop架构

Hadoop 有一个主从拓扑网络，在这个拓扑里面，有一个 master 节点和多个 slave 节点。master 节点的功能就是把任务分配到多个 slave 节点和管理资源。这些 slave 节点是真正运行计算代码和存储数据的机器。而 Master 节点存储的是元数据（即关于数据的数据），元数据包括哪些呢？下面会详细介绍。

Hadoop 架构由3大组件构成：
**HDFS
Yarn
MapReduce**

## HDFS

HDFS 即 Hadoop 分布式文件系统。它为 Hadoop 提供数据存储功能。HDFS 把大的数据文件划分成一个个小的数据单元叫做块（block），并以分布式的方式存储这些块。HDFS 有两个守护进程，一个运行在 master 节点——NameNode，另一个运行在 slave 节点——DataNode。

### NameNode和DataNode

HDFS 也是一种 Master-slave 架构，NameNode 是运行 master 节点的进程，它负责命名空间管理和文件访问控制。DataNode 是运行在 slave 节点的进程，它负责存储实际的业务数据。在 HDFS 内部，一个文件会被划分成许多数据块，并存储在多个 slave 节点。Namenode 管理文件系统命名空间的修改，比如，打开，关闭和重命名文件或者目录这些操作。NameNode 会对数据块与DataNode 的映射关系进行维护和跟踪。DataNode 接收来自文件系统客户端的读写请求。还会根据 NameNode 的指令创建，删除和复制数据块。
![namenode-datanode](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/namenode-datanode_20191001230833407833-20210111222747840.png)

### HDFS的块概念

块是计算机系统最小的存储单元。它是分配给文件的最小的连续存储单元。在 Hadoop，也有块的概念，只是比计算机系统的块要大得多，默认是128MB或者256MB。
![hdfs块概念](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/hdfs-block_20191001230938387642-20210111222755597.png)

在指定 block 大小的时候要非常谨慎。为了解释原因，让我们举个例子，假如一个文件有700MB，如果 block 大小设置为128MB，那么 HDFS 会把这个文件划分成6个 block。5个128MB，一个是60MB。如果 block 的大小4KB，这个时候会出现什么情况呢？在 HDFS 里面，文件大小一般都是TB到PB级别。每个 block 是4KB的话，那么文件会被划分成很多小的 block。这时，NameNode 将会创建大量元数据，这些元数据的量将会塞满NameNode 的内存，导致 NameNode 运行效率低下，甚至奔溃。因此，在选择 HDFS 块大小的时候应该格外小心。

### 副本管理

为了实现数据容错，HDFS 使用了一种复制技术，它会把块复制到不同的 DataNode 节点。副本因子决定了一个数据块可以被复制多少个，默认是3个，其实这个副本因子我们可以配置成任何值。
![hdfs副本管理](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/block-replication_20191001231037441320-20210111222804972.png)

上面这个图展示了 HDFS 的块复制机制。如果我们有一个1G的文件，复制因子是3，那么存储这个文件就需要3G的容量。

为了维护复制因子，NameNode 会从各个 DataNode 收集块报告，当块的副本数大于或者小于复制因子时，NameNode 将会响应的删除和新增块副本。

### HDFS机架感知

![HDFS机架感知](http://www.hadoopdoc.com/media/editor/rack-awareness_20191001231124988263.png)
一个机架都会有很多个 DataNode 机器，在生产环境下，一般都会有好几个这样的机架。HDFS 根据机架感知算法，以分布式方式放置块的副本。机架感知算法具有低延迟和容错特性。假如把复制因子配置为3，那么按照这个算法，HDFS 会把第一个 block 放在本地机架，把另外两个 block 放在另外一个机架的不同 DataNode 节点。HDFS 不会在同一个机架存放超过2个 block。

## MapReduce

MapReduce 是 Hadoop 生态下的海量数据计算框架。利用它可以快速开发出处理海量数据的应用程序，并且应用程序是以并行的方式运行在由价格低廉的机器组成的大规模集群之上的。

MapReduce 任务由 Map 任务和 Reduce 任务组成。每个任务负责计算一部分数据，这样将会把工作负载分发到各个机器上面。Map 任何的主要工作是加载，解析，转换和过滤数据，而 Reduce 任务则是处理来自 Map 任务输出的数据，它会对这些 map 输出的中间数据结果进行分组和聚合操作。

MapReduce 处理的输入文件是存放在 HDFS 上的，如何把文件分割成数据分片取决于文件的输入格式（inputformat），数据分片其实是输入文件数据块的字节形式，map 任何会加载这些数据分片，map 任务会尽量在离数据最近的 datanode 机器上执行。这样数据就不需要通过网络传输，而是在本地就能对数据进行处理，避免了带宽资源的消耗。
![MapReduce数据流向图](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/hadoop-mapreduce-data-flow-execution-1_20191001231203020254-20210111222817592.gif)

### Map任务

Map任务分为以下阶段：

#### RecordReader

RecordReader 会把输入分片转换成记录，它将数据解析为记录，但不解析记录本身。接着把数据以键值对的方式提供给 mapper 函数，一般键是记录的位移信息，而值是具体的数据记录。一个键值对就是一条记录。

#### Map

在这一阶段，用户定义的 mapper 函数会处理来自 recordreader 输出的键值对数据，并输出0个或者多个中间键值对结果。

键值对具体如何被处理取决于你开发的mapper函数，通常情况下，reduce函数会利用键做分组操作，而值会根据键做聚合计算并产生最终的结果。

#### Combiner

Combiner 其实也是 reduce 操作，只是它是在 map 阶段做数据聚合操作，并且这是一种可选的操作，即你可以不使用 Combiner，但是在必要的时候使用它会提高任务执行效率。Combiner 从 mapper 获取中间数据，并对这些数据进行聚合。在很多情况下，使用 Combiner 可以减少网络传输的数据量。例如，传输3次（Hello World, 1）比传输一次（Hello World, 3）要消耗更多的带宽资源。

#### Partitioner

partitioner 从 mapper 拉取键值对数据结果，它会把这些数据分割成分片，每个 reducer 一个分片。默认情况下，partitioner 会对键取哈希值并和reducer 的数量取模运算：key.hashcode() % reducer 数量。这样就可以把键均匀的分发到每个 reducer，还可以确保具有相同 key 的数据落到落在同一个 reducer。被分区的数据将会被写到本地文件系统，等待响应的 reducer 来拉取。

### Reduce任务

reduce 任务分为以下几个阶段：

#### Shuffle和排序

shuffle 和排序操作是 reducer 的起始步骤，该阶段会把 partitioner 输出到文件系统的数据拉取到 reducer 所执行的机器上，并对数据做排序操作形成一个大的数据列表，排序的目的是把相同键的数据聚集在一起，当然，这些操作是由MapReduce框架自动完成的，我们不能对这些操作做改动。但是我们可以通过comparator 对象对键的排序和分区规则进行定制化开发。

#### Reduce

reducer 会对每个的键对应的分组做 reduce 操作。mapreduce 框架把键和一个包含该键的所有值的迭代对象传递给 reduce 函数。

我们可以开发 reducer 来对数据进行不同方式的过滤、聚合以及连接操作。reduce函数执行完之后，它会把生成的0个或者多个键值对传递给 outputformat。

#### OutputFormat

这是最后一步，它接收 reducer 的键值对数据，并通过 recordwrite 把它们写到文件。默认情况下，它是用 tab 分隔符分割键和值，用换行符分割记录。你可以开发其他的 outputformat 来定制修改这个默认规则。但最终的数据都会被写到 HDFS。
![mapreduce工作原理](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/mapreduce-work_20191001231307953395-20210111222828415.png)

### Yarn

Yarn 一个资源管理系统，其作用就是把资源管理和任务调度/监控功分割成不同的进程。Yarn 有一个全局的资源管理器叫 ResourceManager，每个 application 都有一个 ApplicationMaster 进程。一个 application 可能是一个单独的 job 或者是 job 的 DAG （有向无环图）。

在 Yarn 内部有两个守护进程，分别是 ResourceManager 和 NodeManager。ResourceManager 负责给 application 分配资源，而 NodeManager 负责监控容器使用资源情况，并把资源使用情况报告给 ResourceManager。这里所说的资源一般是指CPU、内存、磁盘、网络等。

ApplicationMaster 负责从 ResourceManager 申请资源，并与 NodeManager 一起对任务做持续监控工作。
![Yarn架构图](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/Yarn-Architecture_20191001231338344714-20210111222715674.png)

ResourceManager 有两个比较重要的组件 —— Scheduler 和 ApplicationMaster。

#### Scheduler

Scheduler负责给各种application分配资源，它是纯粹的调度器，因为它并不会跟踪application的状态。它也不会重新对因为软硬件故障而失败的任务进行调度。它只会根据application的要求分配资源。

#### ApplicationManager

ApplicationManager 作用如下：

- 接收任务的提交
- 为 ApplicationMaster 申请第一个 Container。 Container 是 YARN 中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等
- 在 ApplicationMaster 容器失效的时候重启它。

ApplicationMaster 的作用如下：

- 从Scheduler申请资源容器
- 跟踪Container的状态
- 监控application运行进度

我们可以通过 Yarn 的集群联结特性，把节点数量扩展到几千个，这个特性让我们可以把多个 Yarn 集群联结成一个超大的集群。这样就可以把相互独立的集群联结起来，用于执行超大型的任务。

### Yarn的特性

Yarn 具有下面这些特性：

**多租户**
Yarn允许在同样的 Hadoop 数据集使用多种访问引擎。这些访问引擎可能是批处理，实时处理，迭代处理等。

**集群利用率**
在资源自动分配的情况下，跟早期的 Hadoop 版本相比，Yarn 拥有更高的集群利用率。

**可扩展性**
Yarn 可以根据实际需求扩展到几千个节点，多个独立的集群可以联结成一个更大的集群。

**兼容性**
Hadoop 1.x 的 MapReduce 应用程序可以不做任何改动运行在 Yarn 集群上面。