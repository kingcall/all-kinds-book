- - [实时ods 的背景](#实时ods-的背景)
  - 传统离线架构
    - 存在的问题
      - [Sqoop做全量数据同步](#sqoop做全量数据同步)
      - [Sqoop 做增量数据同步](#sqoop-做增量数据同步)
      - [binlog 做还原](#binlog-做还原)
  - Delta Lake
    - 存在的问题
      - 小文件

## 实时ods 的背景

- 实时ods 可以让我们在不改变现有的调度以及sql 任务下，降低数据的延迟
- 不用像实时数仓那样复杂，并且涉及到任务迁移

## 传统离线架构

![image-20201221213113833](/Users/liuwenqiang/Library/Application%20Support/typora-user-images/image-20201221213113833.png)

### 存在的问题

#### Sqoop做全量数据同步

1. 使用Apache Sqoop做全量数据同步，会对业务Mysql库/HDFS造成压力，并且数据延迟大

#### Sqoop 做增量数据同步

1. 使用Apache Sqoop做增量同步，一般只能使用某个时间字段(例如updatetime)来同步新修改的数据。**这样在做分区表时，需要比较复杂的离线合并**，并且数据延迟还是很大

#### binlog 做还原

- 使用binlog 的方式进行数据同步。源库在进行插入、更新、删除等操作的时候会产生binlog，我们只需要将binlog打入KafKa，从 Kafka 中读取 binlog，逐条解析后执行对应的操作即可。但是这种方式要求下游能够支持比较频繁的update/delete操作，以应对上游频繁的 update/delete 情形。这里可以选择KUDU或者HBASE 作为目标存储。
- 但是，由于KUDU和HBASE不是数仓，无法存储全量的数据，所以需要定期把其中的数据导入到Hive中
- 需要注意的是，这种方式存在多个组件运维压力大、Merge逻辑复杂等缺点。

## Delta Lake

- Delta Lake只需要一个streaming sql即可实现实时增量同步,并且会帮你自动做数据还原
- 还是先从binlog到KafKa，与之前的方式不同的是无需将KafKa中的binlog回放到HBASE或者KUDU，而是直接放入Delta Lake即可。这种方案使用方便，无需额外运维，Merge逻辑容易实现，且几乎是一个实时的数据流。

![image-20201221213131133](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/21/21:31:31-image-20201221213131133.png)

- 其本质就是不断的将每一个mini batch给Merge INTO到目标表中。由于 Spark Streaming 的 mini batch 调度建个可以设置在秒级，因此该方案实现了近实时的数据同步。

### 存在的问题

#### 小文件

- 最主要的就是小文件问题，比如每五秒执行一次batch，那么一天就会有非常多的batch，可能产生海量的小文件，严重影响表的查询性能
- 增大调度批次间隔：如果对实时性要求不是很高，可以增大调度批次间隔，减少小文件产生的频率；
- 小文件合并：进行小文件的合并，减小小文件的数量，其语法如下：OPTIMIZE WHERE where_clause]
- 自适应执行：自适应执行可以合并一些小的reduce task，从而减少小文件数量。

> 对于小文件合并的optimize触发我们做了两种方式。

- 第一种是自动化的optimize，就是在每一个mini batch执行完之后都进行检测是否需要进行合并，如果不需要就跳到下一个mini batch，判断的规则有很多，比如小文件达到一定数量、总的文件体积达到一定大小就进行合并，当然在合并的时候也进行了一些优化，比如过滤掉本身已经比较大的文件。自动化的optimize方式每过一定数量的batch就要进行一次merge操作，可能对数据数据摄入造成一定影响
- 因此还有第二种方式，就是定期执行optimize的方式，这种方式对于数据实时摄入没有影响。但是，定期执行optimize的方式会存在事务冲突的问题，也就是optimize与流冲突，对于这种情况我们优化了Delta内部的事务提交机制，让insert流不必失败，如果在optimize之前进行了update/delete，而optimize成功了，那么在成功之后要加一个重试的过程，以免流断掉。