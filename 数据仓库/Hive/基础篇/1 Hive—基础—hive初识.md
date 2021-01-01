[TOC]

## Hive 是什么

### Hive 的定义

那么，到底什么是Hive，我们先看看Hive官网Wiki是如何介绍[Hive](https://cwiki.apache.org/confluence/display/Hive/Home)的

Apache Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张表，并且提供了通过SQL 对存储在分布式中的大型数据集的查询和管理，主要提供以下功能：

1. 它提供了一系列的通过SQL 访问数据的工具，可用来对数据进行提取/转化/加载（ETL）；
2. 对数据各种类型的数据添加schema
3. 可以存查询和分析存储在HDFS（或者HBase）中的大规模数据；
4. 查询可以通过MapReduce、TEZ、SPARK-SQL来完成的（并不是所有的查询都需要MapReduce来完成，比如select * from XXX就不需要)
5. 通过[Hive LLAP](https://cwiki.apache.org/confluence/display/Hive/LLAP), [Apache YARN](https://hadoop.apache.org/docs/r2.7.2/hadoop-yarn/hadoop-yarn-site/YARN.html) and [Apache Slider](https://slider.incubator.apache.org/) 实现次秒级查询检索

**总结为一句话：hive是基于hadoop的数据仓库**，那么数据仓库的概念有是什么呢，这里我们就简单理解为是一个对数据进行查询和管理的软件，所以说Hive是一种建立在Hadoop文件系统上的数据仓库架构，并对存储在HDFS中的数据进行分析和管理；

这里有一句言外之意，那就是对存储在HDFS中的数据进行分析和管理，如果我们不想使用手工也就是MR，那我们就可以开发一个工具，那么这个工具就可以是Hive也可以是其他工具，整个Hive 的架构如下，而SQL 翻译成大数据的任务就发生在Driver 中

更多关于Hive 的架构设计请阅读[Hive的架构设计](https://blog.csdn.net/king14bhhb/article/details/111769279)

![image-20201226210552649](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/26/21:05:53-image-20201226210552649.png)



#### 如何查询和管理数据呢

Hive提供了标准的SQL功能，其中包括了[SQL:2003](https://en.wikipedia.org/wiki/SQL:2003), [SQL:2011](https://en.wikipedia.org/wiki/SQL:2011), 和 [SQL:2016](https://en.wikipedia.org/wiki/SQL:2016)的特性 ，被称为HQL，对于熟悉SQL的用户可以直接利用Hive来查询数据，Hive-SQL 在提供了标准的SQL功能之外，也支持了扩展，**通过实现用户定义的User Defined Functions（UDF）、User Defined Aggregation Functions（UDAF）、User Defined Table Generating Functions（UDTF）**

**HDFS中最关键的一点就是，数据存储HDFS上是没有schema的概念的(schema:相当于表里面有列、字段、字段名称、字段与字段之间的分隔符等，这些就是schema信息)然而HDFS上的仅仅只是一个纯的文本文件而已，那么，没有schema，就没办法使用sql进行查询了啊，因此，在这种背景下，就有问题产生：如何为HDFS上的文件添加Schema信息，如果加上去，是否就可以通过SQL的方式进行处理了呢？于是强大的Hive出现了**，也就是说Hive 是通过给HDFS 上的文件添加上schema 然后通过SQL处理，而这里的SQL处理不同于传统的关系型数据库，因为SQL 最终是是翻译成MR 进行运行的。

当然除了Hive 查询和处理数据之外,还有Hadoop 平台原生的工具MR也就是说 Hive 是基于Hadoop 构建的一套数据仓库分析系统，它提供了丰富的SQL查询方式来分析存储在Hadoop 分布式文件系统中的数据，可以将结构  化的数据文件映射为一张数据库表，并提供完整的SQL查询功能，**可以将SQL语句转换为MapReduce任务进行运行**，通过自己的SQL 去查询分析需  要的内容，这套SQL 简称Hive SQL，**使不熟悉mapreduce 的用户很方便的利用SQL 语言查询，汇总，分析数据。而mapreduce开发人员可以把**  **己写的mapper 和reducer 作为插件来支持Hive 做更复杂的数据分析。**



#### Hive 的设计定位

Hive和传统的关系型数据库有很大的区别，Hive将外部的任务解析成一个MapReduce可执行计划，而启动MapReduce是一个高延迟的一件事，每次提交任务和执行任务都需要消耗很多时间，这也就决定Hive只能处理一些高延迟的应用（如果你想处理低延迟的应用，你可以去考虑一下Hbase）

同时，由于设计的目标不一样，Hive目前还不支持事务；不能对表数据进行修改（不能更新、删除、插入；只能通过文件追加数据、重新导入数据）；不能对列建立索引（但是Hive支持索引的建立，但是不能提高Hive的查询速度。如果你想提高Hive的查询速度，请学习Hive的分区、桶的应用)。

> 其实修改更新操作现在已经可以了，但是需要单独配置，主要是我们很少去用这个功能，更多的是采用覆盖的方式更新

Hive 本来就不是为OLTP设计，它主要作为数仓工具进行数据处理和管理的工具，因为是基于Hadoop 因此有很高的扩展性，当处理能力跟不上的时候，只需要扩展Hadoop 的处理节点即可。



### 大数据中的承担的角色

Hive 在目前的大数据生态中，**主要承担的角色就是数仓工具**，其实说白了就是将SQL翻译成MR 的工具，但是这么说有一定的不准确，因为它现在支持TEZ和Spark 执行引擎，降低了业务开发的门槛，写SQL就可以而不是要去写MR

为什么这里说是数仓工具，而不是数仓呢，数仓是一个业务上的概念或者系统，而Hive 支持其中的一个主要的环节，就是处理数仓数据。

它没有自己的存储，没有自己的处理能力，主要的角色就是承担了一个翻译的角色，因为整个大数据体现很多组件都是有它自己的处理能力、存储能力的，然后将翻译的结果提交给其他大数据处理组件，然后进行数据处理



### 对比Hbase



同时补充一下hive与hbase的联系与区别，因为Hbase 和Hive作为底层都依赖Hadoop 的组件也是我们日常最常用的组件，下面我们看一下它们的异同点
**共同点：**
1.hbase与hive都是架构在hadoop之上的。都是用hadoop作为底层存储

**区别：**

2.Hive是建立在Hadoop之上为了减少MapReduce jobs编写工作的批处理系统，HBase是为了支持弥补Hadoop对实时操作的缺陷的项目 。
3.想象你在操作RMDB数据库，如果是全表扫描，就用Hive+Hadoop,如果是索引访问，就用HBase+Hadoop 。
4.Hive query就是MapReduce jobs可以从5分钟到数小时不止，HBase是非常高效的，肯定比Hive高效的多。
5.Hive本身不存储和计算数据，它完全依赖于HDFS和MapReduce，Hive中的表纯逻辑。
6.hive借用hadoop的MapReduce来完成一些hive中的命令的执行
7.hbase是物理表，不是逻辑表，提供一个超大的内存hash表，搜索引擎通过它来存储索引，方便查询操作。
8.hbase是列存储。
9.hdfs作为底层存储，hdfs是存放文件的系统，而Hbase负责组织文件。
10.hive需要用到hdfs存储文件，需要用到MapReduce计算框架。



## 总结

1. Hive是一个构建在Hadoop上的数据仓库框架。最初，Hive是由Facebook开发，后来移交由Apache软件基金会开发，并作为一个Apache开源项目。是面向分析型应用

2. Hive和传统数据仓库一样，主要用来协助分析报表，支持决策。与传统数据仓库较大的区别是：Hive 可以处理超大规模的数据，可扩展性和容错性非常强。

3. Hive 将所有数据存储在HDFS中，并建立在Hadoop 之上，大部分的查询、计算由MapReduce完成,后面由于Hive 的功能完善使得它不再依赖于MR 执行引擎，可以使用其他引擎  


