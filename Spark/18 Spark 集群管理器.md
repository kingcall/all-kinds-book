# Spark 集群管理器

Spark最主要资源管理方式按排名为Hadoop Yarn, Apache Standalone 和Mesos。在单机使用时，Spark还可以采用最基本的local模式。

目前Apache Spark支持三种分布式部署方式，分别是standalone、spark on mesos和 spark on YARN，其中，第一种类似于MapReduce 1.0所采用的模式，内部实现了容错性和资源管理，后两种则是未来发展的趋势，部分容错性和资源管理交由统一的资源管理系统完成，让Spark运行在一个通用的资源管理系统之上，这样可以与其他计算框架，比如MapReduce，公用一个集群资源，最大的好处是降低运维成本和提高资源利用率（资源按需分配）。

这里将介绍这三种部署方式，并比较其优缺点。

## Standalone模式

即独立模式，自带完整的服务，可单独部署到一个集群中，无需依赖任何其他资源管理系统。从一定程度上说，该模式是其他两种的基础。目前Spark在standalone模式下是没有任何单点故障问题的，这是借助zookeeper实现的，思想类似于Hbase master单点故障解决方案。该模式适合用于集群应用的测试。

## Spark On Mesos模式

这是很多公司采用的模式，官方推荐这种模式（当然，原因之一是血缘关系）。正是由于Spark开发之初就考虑到支持Mesos，因此，目前而言，Spark运行在Mesos上会比运行在YARN上更加灵活，更加自然。目前在Spark On Mesos环境中，用户可选择两种调度模式之一运行自己的应用程序：

### 粗粒度模式（Coarse-grained Mode）

每个应用程序的运行环境由一个Dirver和若干个Executor组成，其中，每个Executor占用若干资源，内部可运行多个Task。应用程序的各个任务正式运行之前，需要将运行环境中的资源全部申请好，且运行过程中要一直占用这些资源，即使不用，最后程序运行结束后，回收这些资源。

举个例子，比如你提交应用程序时，指定使用5个executor运行你的应用程序，每个executor占用5GB内存和5个CPU，每个executor内部设置了5个slot，则Mesos需要先为executor分配资源并启动它们，之后开始调度任务。另外，在程序运行过程中，mesos的master和slave并不知道executor内部各个task的运行情况，executor直接将任务状态通过内部的通信机制汇报给Driver，从一定程度上可以认为，每个应用程序利用mesos搭建了一个虚拟集群自己使用。

### 细粒度模式（Fine-grained Mode）

鉴于粗粒度模式会造成大量资源浪费，Spark On Mesos还提供了另外一种调度模式：细粒度模式，这种模式类似于现在的云计算，思想是按需分配。与粗粒度模式一样，应用程序启动时，先会启动executor，但每个executor占用资源仅仅是自己运行所需的资源，不需要考虑将来要运行的任务，之后，mesos会为每个executor动态分配资源，每分配一些，便可以运行一个新任务，单个Task运行完之后可以马上释放对应的资源。每个Task会汇报状态给Mesos slave和Mesos Master，便于更加细粒度管理和容错，这种调度模式类似于MapReduce调度模式，每个Task完全独立，优点是便于资源控制和隔离，但缺点也很明显，短作业运行延迟大。

## Spark On YARN模式

这是一种很有前景的部署模式。但限于YARN自身的发展，目前仅支持粗粒度模式（Coarse-grained Mode）。这是由于YARN上的Container资源是不可以动态伸缩的，一旦Container启动之后，可使用的资源不能再发生变化，不过这个已经在YARN计划中了。

spark on yarn 的支持两种模式：

- yarn-cluster：适用于生产环境；
- yarn-client：适用于交互、调试，希望立即看到app的输出

yarn-cluster和yarn-client的区别在于yarn appMaster，每个yarn app实例有一个appMaster进程，是为app启动的第一个container；负责从ResourceManager请求资源，获取到资源后，告诉NodeManager为其启动container。yarn-cluster和yarn-client模式内部实现还是有很大的区别。如果你需要用于生产环境，那么请选择yarn-cluster；而如果你仅仅是Debug程序，可以选择yarn-client。

## 总结

上面三种分布式部署方式各有利弊，通常需要根据实际情况决定采用哪种方案。进行方案选择时，往往要考虑公司的技术路线（采用Hadoop生态系统还是其他生态系统）、相关技术人才储备等。上面涉及到Spark的许多部署模式，究竟哪种模式好这个很难说，需要根据你的需求，如果你只是测试Spark Application，你可以选择local模式。而如果你数据量不是很多，Standalone 是个不错的选择。当你需要统一管理集群资源（Hadoop、Spark等），那么你可以选择Yarn或者mesos，但是这样维护成本就会变高。

- 从对比上看，mesos似乎是Spark更好的选择，也是被官方推荐的
- 但如果你同时运行hadoop和Spark,从兼容性上考虑，Yarn是更好的选择。 · 如果你不仅运行了hadoop，spark。还在资源管理上运行了docker，Mesos更加通用。
- Standalone对于小规模计算集群更适合！