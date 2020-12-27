[toc]
## Hive 的架构

我们知道MapReduce和Spark它们提供了高度抽象的编程接口便于用户编写分布式程序，它们具有极好的扩展性和容错性，能够处理超大规模的数据集。这些计算引擎提供了面向高级语言（比如Java，Python等）的编程接口，然而，考虑到分布式程序编写的复杂性，直接使用这些编程接口实现应用系统（比如报表系统）无疑会提高使用门槛，降低开发效率。

考虑到SQL仍然是一种非常主流的数据分析语言，开源社区在分布式计算框架基础山构建了支持SQL的引擎，其中典型的代表是MapReduce之上的Hive以及Spark之上的Spark SQL，这些数据分析引擎通常不仅做到了对标准SQL的支持，还对标准SQL并进行了大量的扩展，其中最主流的数据分析语言为HQL（Hive Query Language）。简而言之，**Hive是对非Java，python等编程者提供了SQL的方式对hdfs数据做MapReduce操作**，也就是将SQL 翻译成

### SQL ON  Hadoop

大数据计算引擎为大规模数据处理提供了解决方案，它们提供了高级编程语言（比如Java，Python等）编程接口，可让程序员很容易表达计算逻辑。但在大数据领域，仅提供对编程语言的支持是不够的，这会降低一些数据分析场景（比如报表系统）下的开发效率，也提高了使用门槛。为了让更多人使用这些大数据引擎分析数据，提高系统开发效率，大数据系统引入了对SQL的支持。

目前构建在Hadoop之上的SQL引擎主要分为两类，基于计算引擎和基于MPP架构

 Hive是构建在分布式计算框架之上的SQL引擎，它重用了Hadoop中的分布式存储系统HDFS/HBase和计算框架MapReduce/Tez/Spark等。Hive是Hadoop生态系统中的重要部分，也是目前应用最广泛的SQL On Hadoop解决方案。

#### 基于计算引擎

这些SQL引擎是在计算引擎基础上构建的，其基本原理是将SQL语聚翻译成分布式应用程序，之后运行在集群中。典型的代表有构建在MapReduce之上的Hive和构建在Spark之上的Spark SQL。这类SQL引擎的特点是具有良好的扩展性和容错性，能够应对海量数据

#### 基于MPP架构

这些SQL引擎是基于MPP架构构建的，其基本原理是将SQL翻译成可分布式执行的任务，采用Volcano风格的计算引擎并处理这些任务，任务之间的数据流动和交换由专门的Exchange运算符完成。典型的代表有Presto和Impala等。这些SQL引擎具有良好的可扩展性，但容错性较差。

### Hive 的架构

Hive对外提供了三种访问方式，包括Web UI，CLI（Client Line Interface）和Thrift协议（支持JDBC/ODBC）,我们可以将这个Hive看成一个CS 的模型，提供用户调用入口的客户端，提供服务的后端(主要包括三个服务组件构成)，也就是下图那个大的虚线框

![image-20201227123311620](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/27/12:33:12-image-20201227123311620.png)

HiveSQL 通过命令行或者客户端提交，经过 Compiler 编译器，运用 MetaStore 中的元数 据进行类型检测和语法分析，生成一个逻辑方案(Logical Plan)，然后通过的优化处理，产生 一个 MapReduce 任务。

需要注意一下的是，如果我们是通过JDBC 或者ODBC 的方式连接的话，我们的HiveServer2 服务会包含Driver 服务，也就像下面这样，也就是上图中那个小的虚线框

![image-20201226204741803](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/26/20:47:42-image-20201226204741803.png)

如果我们的通过shell 客户端连接的话，会在本地启动一个Driver进程，名字叫`org.apache.hadoop.hive.cli.CliDriver`,也就是你启动多个客户端就会有多个`CliDriver` ，所以建议你启动HiveServer2服务，然后使用beeline 客户端连接，其实你启动多个beeline 客户端，也只使用同一个HiveServer2服务，也就是只有一个Driver服务

![image-20201226204958694](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/26/20:49:59-image-20201226204958694.png)



#### 客户端

- shell cli(old cli) 也就是我们用的最多的客户端，它是直接启动了一个进程连接到MetaStore上的
- beeline 它使用的是JDBC 的方式连接到HiveServer(HiveServer1 或者HiveServer2) 上的
- HWI(Hive Web Interface) 几乎没人用
- 其他语言的客户端，其他语言只要连接到HiveServer上就可以对Hive 进行操作，我们可以看到beeline就是Hive 统一访问方式的一次尝试



#### Driver(驱动引擎)

与关系型数据库的查询引擎类似，Driver实现了SQL解析(语法分析、编译、优化)，生成逻辑计划，物理计划，查询优化与执行等，它的输入是SQL语句，输出为一系列分布式执行程序（可以为MapReduce，Tez或Spark等)，如果你学习过JDBC 的话，你一定对Driver 不会感到陌生

Hive 的核心是驱动引擎， 驱动引擎由四部分组成：

1.  解释器：解释器的作用是将 HiveSQL 语句转换为抽象语法树（AST）

 	2.  编译器：编译器是将语法树编译为逻辑执行计划
 	3.  优化器：优化器是对逻辑执行计划进行优化
 	4.  执行器：执行器是调用底层的运行框架执行逻辑执行计划



#### MetaStore

Hive Metastore是管理和存储元数据的服务，**元数据**通俗的讲，就是存储在 Hive 中的数据的描述信息。Hive 中的元数据通常包括：表的名字，表的列和分区及其属性，表的属性（内部表和 外部表），表的数据所在目录

它保存了数据的基本信息以及数据表的定义等，为了能够可靠地保存这些元信息，Hive Metastore一般将它们持久化到关系型数据库中，默认采用了嵌入式数据库Derby（数据存放在内存中），用户可以根据需要启用其他数据库，比如MySQL

客户端连接Metastore服务，Metastore再去连接MySQL数据库来存取元数据。有了Metastore服务，就可以有多个客户端同时连接，而且这些客户端不需要知道MySQL数据库的用户名和密码，只需要连接Metastore 服务即可。 



#### Hadoop

Hive依赖与Hadoop，包括分布式文件系统HDFS，分布式资源管理系统YARN以及分布式计算引擎MapReduce，Hive中的数据表对应的数据存放在HDFS上，计算资源由YARN分配，而计算任务则来自MapReduce引擎。

其实现在的话也可以不依赖分布式计算引擎MapReduce了，因为现在Hive 实现了引擎可插拔的设计，也就是说我们只需要通过配置就可以实现执行引擎的切换

## 总结

1. 今天我们主要讲解了Hive 的架构设计，以及简单介绍了它的一些服务，后面我们会针对每一个服务做单独的讲解，例如MetaStore,HiveServer2 等

2. Hive整体上来说是一个CS 的架构

3. Hive 将SQL 解析成大数据的任务发生在Driver 中，然后将生产的任务交给大数据执行引擎进行执行

   

