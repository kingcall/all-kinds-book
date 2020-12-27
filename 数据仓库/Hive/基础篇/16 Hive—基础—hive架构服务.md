[TOC]

## Hive 服务详解

前面我们学习[数仓工具—Hive的架构设计](https://blog.csdn.net/king14bhhb/article/details/111769279) 的时候学到了很多概念，像元数据服务什么的，其实架构设计里的每一项都对应的是一种服务或者是一个进程，这节我们就学习一下它

### 元数据服务(MetaStore)

#### Metastore 初识

按官网的介绍，不管是使用Hive CLI、客户端还是HWI访问Hive，都需要首先启动Hive 元数据服务，否则无法访问Hive数据库。否则会报异常

```
2020-12-22 21:36:12,203 INFO  [1667ccc9-b01a-4f12-af7f-6ad7a563f46e main] metastore.HiveMetaStoreClient (HiveMetaStoreClient.java:open(441)) - Trying to connect to metastore with URI thrift://localhost:9083
2020-12-22 21:36:12,224 WARN  [1667ccc9-b01a-4f12-af7f-6ad7a563f46e main] metastore.HiveMetaStoreClient (HiveMetaStoreClient.java:open(526)) - Failed to connect to the MetaStore Server...
2020-12-22 21:36:12,224 INFO  [1667ccc9-b01a-4f12-af7f-6ad7a563f46e main] metastore.HiveMetaStoreClient (HiveMetaStoreClient.java:open(557)) - Waiting 1 seconds before next connection attempt.
2020-12-22 21:36:13,227 INFO  [1667ccc9-b01a-4f12-af7f-6ad7a563f46e main] metastore.HiveMetaStoreClient (HiveMetaStoreClient.java:open(441)) - Trying to connect to metastore with URI thrift://localhost:9083
2020-12-22 21:36:13,228 WARN  [1667ccc9-b01a-4f12-af7f-6ad7a563f46e main] metastore.HiveMetaStoreClient (HiveMetaStoreClient.java:open(526)) - Failed to connect to the MetaStore Server...
2020-12-22 21:36:13,228 INFO  [1667ccc9-b01a-4f12-af7f-6ad7a563f46e main] metastore.HiveMetaStoreClient (HiveMetaStoreClient.java:open(557)) - Waiting 1 seconds before next connection attempt.
2020-12-22 21:36:14,229 INFO  [1667ccc9-b01a-4f12-af7f-6ad7a563f46e main] metastore.HiveMetaStoreClient (HiveMetaStoreClient.java:open(441)) - Trying to connect to metastore with URI thrift://localhost:9083
2020-12-22 21:36:14,230 WARN  [1667ccc9-b01a-4f12-af7f-6ad7a563f46e main] metastore.HiveMetaStoreClient (HiveMetaStoreClient.java:open(526)) - Failed to connect to the MetaStore Server...
2020-12-22 21:36:14,230 INFO  [1667ccc9-b01a-4f12-af7f-6ad7a563f46e main] metastore.HiveMetaStoreClient (HiveMetaStoreClient.java:open(557)) - Waiting 1 seconds before next connection attempt.
2020-12-22 21:36:15,235 WARN  [1667ccc9-b01a-4f12-af7f-6ad7a563f46e main] metadata.Hive (Hive.java:registerAllFunctionsOnce(277)) - Failed to register all functions.
java.lang.RuntimeException: Unable to instantiate org.apache.hadoop.hive.ql.metadata.SessionHiveMetaStoreClient
	at org.apache.hadoop.hive.metastore.utils.JavaUtils.newInstance(JavaUtils.java:86)
	at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.<init>(RetryingMetaStoreClient.java:95)
	at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.getProxy(RetryingMetaStoreClient.java:148)
	at org.apache.hadoop.hive.metastore.RetryingMetaStoreClient.getProxy(RetryingMetaStoreClient.java:119)
	at org.apache.hadoop.hive.ql.metadata.Hive.createMetaStoreClient(Hive.java:4299)
	at org.apache.hadoop.hive.ql.metadata.Hive.getMSC(Hive.java:4367)
	at org.apache.hadoop.hive.ql.metadata.Hive.getMSC(Hive.java:4347)
	at org.apache.hadoop.hive.ql.metadata.Hive.getAllFunctions(Hive.java:4603)
	at org.apache.hadoop.hive.ql.metadata.Hive.reloadFunctions(Hive.java:291)
```

这里我们可以看到在几次尝试之后，终于还是报错了

这里我们可以尝试将这个服务启动然后进行连接hive	 `nohup hive --service metastore &`

```
2020-12-22 21:39:06,432 INFO  [38c08100-6fd8-499d-9c8e-578d890ab8e4 main] CliDriver (SessionState.java:printInfo(1227)) - Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
2020-12-22 21:39:06,438 INFO  [pool-7-thread-1] session.SessionState (SessionState.java:createPath(790)) - Created HDFS directory: /tmp/hive/liuwenqiang/ac02bd92-da32-4a9b-a286-580b0f4e1e1b
2020-12-22 21:39:06,443 INFO  [pool-7-thread-1] session.SessionState (SessionState.java:createPath(790)) - Created local directory: /tmp/hive/local/ac02bd92-da32-4a9b-a286-580b0f4e1e1b
2020-12-22 21:39:06,445 INFO  [pool-7-thread-1] session.SessionState (SessionState.java:createPath(790)) - Created HDFS directory: /tmp/hive/liuwenqiang/ac02bd92-da32-4a9b-a286-580b0f4e1e1b/_tmp_space.db
2020-12-22 21:39:06,491 INFO  [pool-7-thread-1] metadata.HiveMaterializedViewsRegistry (HiveMaterializedViewsRegistry.java:run(171)) - Materialized views registry has been initialized
```

这个时候你就看不到不错了，hive 命令行就可以成功连上了

元数据包含用Hive创建的database、table等的元信息。元数据存储在关系型数据库中。如Derby、MySQL等。如果你是用的mysql 的话，hive的元数据存在于mysql中，在mysql中会有一个hive库，存放相应的表，一共53张表 每张表都有其具体的作用，如有需要可自行上网百度，这里就不介绍了



#### Metastore作用

**客户端连接metastore服务，metastore再去连接MySQL数据库来存取元数据。有了metastore服务，就可以有多个客户端同时连接，而且这些客户端不需要知道MySQL数据库的用户名和密码，只需要连接metastore 服务即可。** 



#### Metastore 服务的三种方式

##### 默认开启方式

 没有配置metaStore的时候,每当开启bin/hive;或者开启hiveServer2的时候,都会在内部启动一个metastore，嵌入式服务;资源比较浪费,如果开启多个窗口,就会存在多个metastore server。

**hive服务和metastore服务运行在同一个进程中**，derby服务也运行在该进程中.内嵌模式使用的是内嵌的Derby数据库来存储元数据，也不需要额外起Metastore服务。

##### local mataStore(本地)

当metaStore和装载元数据的数据库(MySQL)存在**同一机器上时配置是此模式**, 开启metastore服务就只需要开启一次就好,避免资源浪费!

##### Remote Metastore(远程)

当metaStore和装载元数据的数据库(MySQL)不存在同一机器上时配置是此模式,开启metastore服务就只需要开启一次就好,避免资源浪费!

### Hive Web Interface(HWI)

HWI 是 Hive 的Web 接口，为什么说是接口而不是界面呢，那是因为它和hive 命令行一样，可以做很多操作，而不单单是显示界面,Hive Web Interface 缩写为HWI,Hive Web界面是Hive 命令行的替代产品，使用web界面是开始使用Hive的一个很好的方法。但是我们知道它并没有替代掉Hive 的命令行，我们也知道不止它没有成功，甚至后来的Beeline 也没有做到，反而是Hive Web Interface 自从 Hive [2.2.0](https://issues.apache.org/jira/browse/HIVE-15622) 的时候被移除了

所以我们知道有这么个东西就行了，后面我也不打算对它多做介绍了



### Hiveserver2

HiveServer是一种可选服务，允许远程客户端可以使用各种编程语言向Hive提交请求并检索结果。HiveServer是建立在Apache ThriftTM（[http://thrift.apache.org/）](https://link.zhihu.com/?target=http%3A//thrift.apache.org/%EF%BC%89) 之上的，因此有时会被称为Thrift Server，这可能会导致混乱，因为新服务HiveServer2也是建立在Thrift之上的．自从引入HiveServer2后，HiveServer也被称为HiveServer1。

HiveServer无法处理来自多个客户端的并发请求.这实际上是HiveServer导出的Thrift接口所施加的限制，也不能通过修改HiveServer源代码来解决。

HiveServer2对HiveServer进行了重写，来解决这些问题，从Hive 0.11.0版本开始。建议使用HiveServer2。



![image-20201222214721249](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/13:29:58-21:47:21-image-20201222214721249.png)

整个Hiveserver2 在hive 的架构中承担的角色就是，其实以JDBC 或者ODBC 的方式供其他语言调用Hive,更多关于Hiveserver2的请看[Hive架构之HiveServer2](https://blog.csdn.net/king14bhhb/article/details/111770337)



#### beeline

 beeline 是 hive 提供的一个**新的命令行工具**，**基于SQLLine CLI的JDBC客户端**，beeline 要与HiveServer2配合使用，支持嵌入模式和远程模式两种，也即既可以像hive client一样访问本机的hive服务，也可以通过指定ip和端口远程访问某个hive服务。hive 官网是推荐使用beeline，它还提供了更为友好的显示方式（类似MySQL client）

a、要使用 beeline ，先把 hiveserver2 启动起来，默认端口为10000

```bash
# 启动 hiveserver2
$ hiveserver2
```

b、使用beeline

```bash
# 1、指定要连接的hiveserver2的主机、端口
beeline -u jdbc:hive2://hd1:10000
# 2、如果是本机的hiveserver2，则可省略主机、端口
```

### HCatalog

 HCatalog是Hadoop的元数据和数据表的管理系统，它是基于Hive的元数据层，通过类SQL的语言展现Hadoop数据的关联关系，支持Hive、Pig、MapReduce等共享数据和元数据，使用户在编写应用程序时无需关心数据是怎么存储、存在哪里，避免用户因schema和存储格式的改变而受到影响。HCatalog的这种灵活性，使得在不影响到使用者的应用程序读取数据的情况下，数据产生者可以在数据中增加新列。在不影响生产者或使用者的情况下，管理员可以迁移数据或是改变数据的存储格式

![image-20201221145711007](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20201221145711007.png)

 从上图可看出，HCatalog低层支持多种文件格式的数据存储方法，上层支持Pig、MapReduce、Hive、Streaming等多种应用。

这样的好处在于，可以支持不同的工具集和系统能在一起使用，例如数据分析团队，一开始可能只使用一种工具（如Hive，Pig，Map Reduce），而随着数据分析工作的深入，需要多种工具相结合，如刚开始使用Hive进行分析查询的用户，后面还需要使用Pig为ETL过程处理或建立数据模型；刚开始使用Pig的用户发现，他们更想使用Hive进行分析查询。在这些情况下，通过HCatalog提供了元数据之间的共享，使用户更方便的在不同工具间切换操作，比如在 Map Reduce或Pig中载入数据并进行规范化，然后通过Hive进行分析，当这些工具都共享一个metastore时，各个工具的用户就能够即时访问其他工具创建的数据，而无需载入和传输的步骤，非常高效、方便。

 Apache Hive 对 **[HCatalog 有详细的介绍说明](https://cwiki.apache.org/confluence/display/Hive/HCatalog+UsingHCat)**。从 hive 0.11.0 版本之后，hive 安装包便提供了 hcatalog，也即安装hive后，里面就已经有了hcatalog了（**[官网说明](https://cwiki.apache.org/confluence/display/Hive/HCatalog+InstallHCat#HCatalogInstallHCat-HCatalogInstalledwithHive)**）接下来将介绍如何配置使用 HCatalog



### **WebHCat**

WebHCat是为HCatalog提供REST API的服务，自hive 0.11.0 版本之后，hive 中也自带了 webhcat （**[官网介绍说明](https://cwiki.apache.org/confluence/display/Hive/WebHCat+InstallWebHCat)**），如下图，通过WebHCat，程序能够通过REST的API很安全的链接和操作HCatalog提供的服务，方便Hive、Pig、MapReduce等应用使用。（类似于通过WebHDFS以web的方式来操作HDFS）

![img](https://static.oschina.net/uploads/space/2017/0628/090624_rclF_876354.png)

使用以下命令启动 webhcat，默认的web端口为50111（须先启动 hcat_srever.sh start）

```bash
webhcat_server.sh start &
```

在浏览器输入 http://172.17.0.1:50111/templeton/v1/status 可查看 hcatalog 的状态，

## 总结

1. Hive有三种使用方式——CLI命令行，HWI（hie web interface）浏览器 以及 Thrift客户端连接方式。

2. Hive 作为大数据中的一员，有很多优秀的插拔式设计可以学习，可扩展的HCatalog，可切换的执行引擎，多语言访问的HiveServer2

   