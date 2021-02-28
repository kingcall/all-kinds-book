[toc]
## presto

- Presto采用典型的master-slave模型

![image-20210222114357179](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210222114357179.png)



![image-20210222114412792](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210222114412792.png)

### 背景
- 它最初是Facebook为数据分析师设计和开发的，用于在 Apache Hadoop 中的大型数据仓库上运行交互式查询。**这意味着，它可以直接对HDFS上的数据进行分析，这是一个非常有用的特性**。
- 在大数据概念出现以前，数据立方体通常通过事实表 + 维度表，也就是星型模型的方式呈现 这种方式的数据冗余性很低，但是每次进行分析都需要连接操作，可以说是用时间换空间。
- Presto 可以直接操作 HDFS 上的数据，也就无需将数据立方体再次导出，少了很多工作量。而传统的数据集市往往基于数据库，这样就需要将数据再次导出到数据库中

### 架构
每个Presto server既是一个coordinator也是一个worker。但是在大型集群中，处于性能考虑，建议单独用一台机器作为coordinator。

#### coordinator

- coordinator 中心的查询角色 接收查询请求、解析SQL 生成执行计划 任务调度 worker管理
- coordinator进行是presto集群的master进程
- coordinator(master)负责meta管理,worker管理，query的解析和调度

#### worker
- worker则负责计算和读写。

#### connector
- presto以插件形式对数据存储层进行了抽象，它叫做连接器，不仅包含Hadoop相关组件的连接器还包括RDBMS连接器
- 具体访问哪个数据源是通过catalog 中的XXXX.properties文件中connector.name决定的提取数据 负责实际执行查询计划

#### discovery service
- 将coordinator和worker结合在一起服务；
- worker节点启动后向discovery service服务注册,coordinator通过discovery service获取注册的worker节点,coordinator便知道在我的集群中有多少个worker能够给我工作，然后我分配工作到worker时便有了根据
- discovery server， 通常内嵌于coordinator节点中，也可以单独部署，用于节点心跳。


### 数据模型
> presto采取三层表结构：

- catalog 对应某一类数据源，例如hive的数据，或mysql的数据
- schema 对应mysql中的数据库/相当于MySQL的database
- table 对应mysql中的表

### 数据源
presto 支持跨数据源连表查询,如 join mysql 和 hive 表

![image-20210222115240246](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210222115240246.png)

- Presto 是一种用于大数据的高性能分布式SQL查询引擎，其架构允许用户查询各种数据源，如 Hadoop、AWS S3、Alluxio、MySQL、Cassandra、Kafka 和 MongoDB。你甚至可以在单个查询中查询来自多个数据源的数据。
- Presto 是Apache 许可证下发布的社区驱动的开源软件。**它是 Dremel 的实现，与 ORC 配合通常会有非常好的效果**。从前面的课时内容可以得知，得益于 Dremel 架构，**Presto 是非常好的 OLAP 与 Ad-hoc 查询工具，并且支持 JDBC**。

Hadoop/Hive connector与存储格式：

HDFS，ORC，RCFILE，Parquet，SequenceFile，Text

开源数据存储系统：

MySQL & PostgreSQL，Cassandra，Kafka，Redis

其他：

MongoDB，ElasticSearch，HBase

### SQL运行过程
> 由客户端提交查询，从Presto命令行CLI提交到coordinator。 coordinator进行解析，分析并执行查询计划，然后分发处理队列到worker。
1. coordinator接到SQL后，通过SQL语法解析器把SQL语法解析变成一个抽象的语法树AST，只是进行语法解析如果有错误此环节暴露
2. 语法符合SQL语法，会经过一个逻辑查询计划器组件，通过connector查询metadata中schema列名列类型等，将之与抽象语法数对应起来，生成一个物理的语法树节点 如果有类型错误会在此步报错
3. 如果通过，会得到一个逻辑的查询计划，将其分发到分布式的逻辑计划器里，进行分布式解析，最后转化为一个个task
4. 在每个task里面，会将位置信息解析出来，交给执行的plan，由plan将task分给worker执行

![image-20210222115305762](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210222115305762.png)





![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1170667-20170622090345101-1073142887.png)

### 低延迟原理

- 基于内存的并行计算
- 流水式计算作业
- 动态编译执行计划
- GC控制

#### MapReduce vs Presto

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/1170667-20170622090852491-2081719078.png)

### 不足
- 不适合多个大表的join操作，因为presto是基于内存的，太多数据内存放不下的如果一个presto查询查过30分钟，那
就kill吧，说明不适合 也违背了presto的实时初衷
- coordinator 和discovery service 的单点故障问题还没有解决
- 对于query是没有容错的，一旦worker挂了，query就执行失败了，与其在这里容错不如直接执行
- **presto 是不支持insert 操作的，只支持查询**

### 对比

#### impala 
- 其实presto 的性能不如impala 好(impala 稍微)，但是presto 支持的数据源非常多

## json

```
select
  json_extract(event_param,'$.poster'),
  app_id,
  event_id,
  stat_date
from
  dw.dwd_track_event_di
where
  pt = '2019-10-29'
  and stat_date = '2019-10-21'
  and event_param is not null
  and json_extract(event_param,'$.poster') is not null
```
