# Spark Streaming 教程

## Spark Streaming是什么

Spark Streaming 是个批处理的流式（实时）计算框架。其基本原理是把输入数据以某一时间间隔批量的处理，当批处理间隔缩短到秒级时，便可以用于处理实时数据流。支持从多种数据源获取数据，包括Kafk、Flume、Twitter、ZeroMQ、Kinesis以及TCP sockets，从数据源获取数据之后，可以使用诸如map、reduce、join等高级函数进行复杂算法的处理。最后还可以将处理结果存储到文件系统，数据库等数据持久化系统。

Spark Streaming处理的数据流介绍如下：

1、Spark Streaming接收Kafka、Flume、HDFS和Kinesis等各种来源的实时输入数据，进行处理后，处理结果保存在HDFS、Databases等各种地方。
![spark streaming流程](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1571322477000_20191017222800838655.png)
2、Spark Streaming接收这些实时输入数据流，会将它们按批次划分，然后交给Spark引擎处理，生成按照批次划分的结果流。
![spark streaming 流处理](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1571322514000_20191017222836564267.png)
3、Spark Streaming提供了表示连续数据流的、高度抽象的被称为离散流的DStream。DStream本质上表示RDD的序列。任何对DStream的操作都会转变为对底层RDD的操作。
![spark Dstream](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1571322558000_20191017222921208067.png)
4、Spark Streaming使用数据源产生的数据流创建DStream，也可以在已有的DStream上使用一些操作来创建新的DStream。
![spark Dstream](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1571322590000_20191017222953780166.png)

## Spark Streaming能做什么

目前而言Spark Streaming主要支持以下三种业务场景：

### 无状态操作

只关注当前批次中的实时数据，例如：

- 商机标题分类，分类http请求端 -> kafka -> Spark Streaming -> http请求端Map -> 响应结果
- 网库Nginx访问日志收集，flume->kafka -> Spark Streaming -> hive/hdfs
- 数据同步，网库主站数据通过“主站”->kafka->Spark Streaming -> hive/hdfs

### 有状态操作

对有状态的DStream进行操作时，需要依赖之前的数据。除了当前新生成的小批次数据，但还需要用到以前所生成的所有的历史数据。新生成的数据与历史数据合并成一份流水表的全量数据，例如:

- 实时统计网库各个站点总的访问量。
- 实时统计网库每个商品的总浏览量，交易量，交易额。

### 窗口操作

定时对指定时间段范围内的DStream数据进行操作，例如：

- 网库主站的恶意访问、爬虫，每10分钟统计30分钟内访问次数最多的用户。

## Spark Streaming 优缺点

与传统流式框架相比，Spark Streaming 最大的不同点在于它对待数据是粗粒度的处理方式，即一次处理一小批数据，而其他框架往往采用细粒度的处理模式，即依次处理一条数据。Spark Streaming 这样的设计实现既为其带来了显而易见的优点，又引入了不可避免的缺点。

### 优点

- Spark Streaming 内部的实现和调度方式高度依赖 Spark 的 DAG 调度器和 RDD，这就决定了 Spark Streaming 的设计初衷必须是粗粒度方式的，同时，由于 Spark 内部调度器足够快速和高效，可以快速地处理小批量数据，这就获得准实时的特性。
- Spark Streaming 的粗粒度执行方式使其确保“处理且仅处理一次”的特性，同时也可以更方便地实现容错恢复机制。
- 由于 Spark Streaming 的 DStream 本质是 RDD 在流式数据上的抽象，因此基于 RDD 的各种操作也有相应的基于 DStream 的版本，这样就大大降低了用户对于新框架的学习成本，在了解 Spark 的情况下用户将很容易使用 Spark Streaming。
- 由于 DStream 是在 RDD 上的抽象，那么也就更容易与 RDD 进行交互操作，在需要将流式数据和批处理数据结合进行分析的情况下，将会变得非常方便。

### 缺点

- Spark Streaming 的粗粒度处理方式也造成了不可避免的延迟。在细粒度处理方式下，理想情况下每一条记录都会被实时处理，而在 Spark Streaming 中，数据需要汇总到一定的量后再一次性处理，这就增加了数据处理的延迟，这种延迟是由框架的设计引入的，并不是由网络或其他情况造成的。