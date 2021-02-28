[toc]
## Impala
- Impala 是 Cloudera 公司推出，可以使用 SQL 对 HDFS、HBase 数据进行高性能、低延迟交互式查询。Impala 是分布式的大规模并行计算（MPP）数据库计算引擎。
- Impala与Apache Hive使用相同的元数据、SQL语法（HiveSQL）、ODBC驱动程序和用户界面（Hue），为批处理或实时查询提供了一个熟悉且统一的平台，Hive用户可以低成本地开销使用 Impala。
- 它是由Java和C++实现的，Java提供的查询交互的接口和实现，C++实现了查询引擎部分，除此之外，Impala还能够共享Hive Metastore，甚至可以直接使用Hive的JDBC jar和beeline等直接对Impala进行查询、支持丰富的数据存储格式（Parquet、Avro等）。

![image-20210222115625950](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210222115625950.png)

- Impala 中包含三种角色，分别是 Impala Daemon、Statestore 和 Catalog Service。Impala集群包含一个 Catalog Server (Catalogd)、一个 Statestore Server (Statestored) 和若干个 Impala Daemon (Impalad)

### Catalogd
Catalogd 主要负责元数据的获取和 DDL 的执行
### Statestored
Statestored 主要负责消息/元数据的广播
### Impalad
Impalad 主要负责查询的接收和执行

![image-20210222115718886](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210222115718886.png)

#### Query Planner 进行 sql 解析，规划执行计划

#### Query Coordinator 管理元数据缓存，负责查询的调度、将任务分配给 Executor

#### Query Executor 负责数据的读取和计算（即：真正干活的角色）

## Impala 查询执行流程
![image-20210222115747889](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210222115747889.png)
- 客户端发送 SQL 查询请求到任其中一台 Impalad 的Query Planner
- 由 Query Planner 解析 SQL，规划执行计划，将任务发给 Query Coordinator
- 由 Query Coordinator 来调度分配任务到相应的 Impalad 节点
- 各 Impalad 节点的 Query Executor 进行执行 SQL工作
- 各 Impalad 节点执行SQL结束以后，将结果返回给 Query Coordinator，Coordinator进行结果汇总
- Query Coordinator 将结果返回给Client

## 合理的部署 Impala 各角色
![image-20210222115812575](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210222115812575.png)
### Impalad 与 DataNode 混合部署
- Impala 常用于统计 HDFS 上的数据，因此强烈建议将 Impalad 和 DataNode 部署到同一台机器。这样可以保证 Impala 任务运行时可以直接从服务器本地读取数据，减少数据的网络传输。
- 如果 Impalad 与 DataNode 分开部署，那么每次进行统计时，都需要先将数据从 DataNode 拉取到 Impalad，再由 Impalad 的 Executor 进行计算，大数据计算中网络 IO 很容易成为瓶颈。

### Impalad 的 Coordinator 与 Executor 独立部署
#### Impalad 支持的部署模式
- Impala 2.9 及以上版本，Impalad 可配置为 Coordinator only、 Executor only 或 Coordinator and Executor（默认）三种模式。默认情况下，每个 Impalad 都包含了 Coordinator 和 Executor。
- 如果 Impalad 配置为 Coordinator only，说明该 Impalad 仅用于协调，不用于真正的计算
- 如果 Impalad 配置为 Executor only，说明该 Impalad 仅用于计算，不用于协调
- 如果 Impalad 配置为 Coordinator and Executor，说明该 Impalad 既用于计算，也用于协调

#### Coordinator 缓存元数据的问题
- Impala 的元数据缓存在catalogd和各个 Coordinator 角色的Impalad中。Catalogd 中的缓存是最新的，各个Coordinator都缓存的是Catalogd内元数据的一个复本。**元数据由Catalogd向外部系统获取，并通过 Statestored 传播给各个 Coordinator**。

- 如果集群中所有的 Impalad 都包含 Coordinator 角色，那么每个 Impalad 节点都需要缓存一份元数据，则每个 Coordinator 节点都需要消耗一定的内存空间。Impala通过Statestored将元数据传播给各个Coordinator，**同步元数据就会消耗特别大的网络带宽，Statestored 的网络 IO 可能成为瓶颈**。

- 笔者运维的 Impala 集群包含 200 个 Impalad 节点，且 Impala 中维护的数据表比较多，Impalad 配置为 Coordinator and Executor 模式，发现 Statestored 的网络 IO 能达到 1GB/s 左右，真是太恐怖了。
- 由于集群中表比较大，元数据比较多，每个 Coordinator 需要占用 20G 左右的内存资源，而且属于 7*24 小时常驻内存。200 个 Impalad 节点，因此 Coordinator 占用了 4T 的内存资源（太可怕了，如果一台服务器100G，相当于 40 台服务器被浪费了）。
- 集群中应该提供更多的资源给 Executor 去干活，应该部署少量的 Coordinator，少量的 Coordinator 即可满足集群的工作负载。并不需要每个 Impalad 都部署 Coordinator，强烈建议生产环境 Coordinator 与 Executor 独立部署，即：选择一些节点单独部署 Executor，再选择一些节点单独部署 Coordinator。
> 问题来了，到底分别部署多少个 Executor 和 Coordinator 呢？**Impala 官方建议每 50 个 Executor 对应一个 Coordinator**。当然这只是一个经验值，具体可以参考角色负载情况进行调整。详细信息请参考 Impala 官网 How to Configure Impala with Dedicated Coordinators[2]，该链接详细讲述了为什么要配置专属的 Coordinator，即为什么要独立部署？且讲述了该如何来控制 Executor 和 Coordinator 的比例。


## 不足
- Impala不支持的地方，例如：不支持update、delete操作，不支持Date数据类型，不支持ORC文件格式等等，而Presto则基本没有这些局限问题（本次测试中基本没有发现）

## 对比
- SparkSQL是Hadoop中另一个著名的SQL引擎，它以Spark作为底层计算框架，Spark使用RDD作为分布式程序的工作集合，它提供一种分布式共享内存的受限形式。 在分布式共享内存系统中，应用可以向全局地址空间的任意位置进行读写操作，而RDD是只读的，对其只能进行创建、转化和求值等操作。这种内存操作大大提高了计算速度。 SparkSQL的性能相对其他的组件要差一些，多表单表查询性能都不突出。
- Impala官方宣传其计算速度是一大优点，在实际测试中我们也发现它的多表查询性能和presto差不多，但是单表查询方面却不如presto好。 而且Impala有很多不支持的地方，例如：不支持update、delete操作，不支持Date数据类型，不支持ORC文件格式等等，所以我们查询时采用Parquet格式进行查询，而且Impala在查询时占用的内存很大。
- Presto综合性能比起来要比其余组件好一些，无论是查询性能还是支持的数据源和数据格式方面都要突出一些，在单表查询时性能靠前，多表查询方面性能也很突出。 由于Presto是完全基于内存的并行计算，所以Presto在查询时占用的内存也不少，但是发现要比Impala少一些，比如多表Join需要很大的内存，Impala占用的内存比Presto要多。
- HAWQ 吸收了先进的基于成本的 SQL 查询优化器，自动生成执行计划，可优化使用Hadoop 集群资源。 HAWQ 采用 Dynamic Pipelining 技术解决这一关键问题。Dynamic Pipelining 是一种并行数据流框架，利用线性可扩展加速Hadoop查询，数据直接存储在HDFS上，并且其SQL查询优化器已经为基于HDFS的文件系统性能特征进行过细致的优化。 但是我们发现HAWQ在多表查询时比Presto、Impala差一些；而且不适合单表的复杂聚合操作，单表测试性能方面要比其余四种组件差很多，HAWQ环境搭建也遇到了诸多问题。
- ClickHouse 作为目前所有开源MPP计算框架中计算速度最快的，它在做多列的表，同时行数很多的表的查询时，性能是很让人兴奋的，但是在做多表的Join时，它的性能是不如单宽表查询的。 **性能测试结果表明ClickHouse在单表查询方面表现出很大的性能优势，但是在多表查询中性能却比较差，不如Presto和Impala、HAWQ的效果好**。
- Greenplum作为关系型数据库产品，它的特点主要就是查询速度快，数据装载速度快，批量DML处理快。 而且性能可以随着硬件的添加，呈线性增加，拥有非常良好的可扩展性。因此，它主要适用于面向分析的应用。 比如构建企业级ODS/EDW，或者数据集市等，Greenplum都是不错的选择。
- 此外我们还对Flink进行了调研发现，Flink 核心是个流式的计算引擎，通过流来模拟批处理，Flink SQL还处于早期开发阶段，未来社区计划通过提供基于REST的SQL客户端，目前SQL客户端不能直接访问Hive，通过YAML file文件定义外部数据源，可以连接文件系统和Kafka，目前短时间我们的SQL测试不太好模拟。所以没有对Flink进行测试分析。

![image-20210222115827650](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210222115827650.png)