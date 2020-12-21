![image-20201221212817997](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/21/21:28:18-image-20201221212817997.png)

- 随着数据多样性的发展，数据仓库这种提前规定schema的模式显得越来难以支持灵活的探索&分析需求，这时候便出现了一种数据湖技术，即把原始数据全部缓存到某个大数据存储上，后续分析时再根据需求去解析原始数据。简单的说，**数据仓库模式是schema on write，数据湖模式是schema on read**

> 早先大家用的都是一些比较成熟的数据仓库系统，数据通过 ETL 导入到数仓。数仓的典型用途是用于 BI 报表之类的分析场景，**场景比较有限**。在移动互联网时代，数据来源更加丰富多样，数据结构也不仅仅是结构化数据，数据用途也不仅限于分析，于是出现了数据湖。数据先不做，或者仅做简单的处理导入到数据湖，然后再进行筛选、过滤、转换等 transform 操作，**于是数仓时代的 ETL 变成了数据湖时代的 ELT**。

- Delta Lake 是一个存储层，为ApacheSpark和大数据workloads提供ACID事务能力，其通过写和快照隔离之间的乐观并发控制（optimisticconcurrencycontrol），在写入数据期间提供一致性的读取，从而为构建在 HDFS 和云存储上的数据湖（data lakes）带来可靠性。Delta Lake 还提供内置数据版本控制，以便轻松回滚
- 随着公司开始从许多不同源收集大量数据，架构师开始**构想一个单一的系统来容纳不同分析产品和工作负载的数据**。大约十年前，公司开始构建数据湖：各种格式原始数据的存储库。数据湖虽然适合存储数据，但缺少一些关键功能：不支持事务、无法提高数据质量、缺乏一致性/隔离性，导致几乎不可能混合处理追加（append）和读取，批处理和流处理作业。由于这些原因，数据湖之前的许多承诺尚未实现，在许多情况下还会失去数据仓库的许多好处。

### 传统的sqoop

- 大数据场景下我们常用的工具是Sqoop，它是一个批处理模式的工具，我们可以用它把业务库中的数据导入到数据仓库。需要注意的时候我们在导入之前要在业务库中的数据中选出能反映时间变化的字段，然后依据时间戳将发生变化的数据导入数据仓库中，这是使用它的一个限制
- 对源库产生压力；
- 延迟大，依赖于调用它的频次；
- 无法应对schema变动，一旦源库中的scheme发生变化，就在对数仓中的表模型重新建模和导入。

### binlog

- 除了使用 sqoop，还有一种方式是使用binlog 的方式进行数据同步。源库在进行插入、更新、删除等操作的时候会产生binlog，我们只需要将binlog打入KafKa，从 Kafka 中读取 binlog，逐条解析后执行对应的操作即可。但是这种方式要求下游能够支持比较频繁的update/delete操作，以应对上游频繁的 update/delete 情形。
- 这里可以选择KUDU或者HBASE 作为目标存储。但是，由于KUDU和HBASE不是数仓，无法存储全量的数据，所以需要定期把其中的数据导入到Hive中，如下图所示。需要注意的是，这种方式存在多个组件运维压力大、Merge逻辑复杂等缺点。

![image-20201221212832708](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/21/21:28:33-image-20201221212832708.png)

## 数据湖

- 现在很多公司内部数据架构中都存在数据湖，数据湖是一种大型数据存储库和处理引擎。它能够存储大量各种类型的数据，拥有强大的信息处理能力和处理几乎无限的并发任务或工作的能力

### 存在的问题

#### 数据湖的读写是不可靠的。

- 数据工程师经常遇到不安全写入数据湖的问题，导致读者在写入期间看到垃圾数据。他们必须构建方法以确保读者在写入期间始终看到一致的数据。

#### 数据湖中的数据质量很低

- 将非结构化数据转储到数据湖中是非常容易的。但这是以数据质量为代价的。没有任何验证模式和数据的机制，导致数据湖的数据质量很差。因此，努力挖掘这些数据的分析项目也会失败。

#### 随着数据的增加，处理性能很差。

- 随着数据湖中存储的数据量增加，文件和目录的数量也会增加。处理数据的作业和查询引擎在处理元数据操作上花费大量时间。在有流作业的情况下，这个问题更加明显。

#### 数据湖中数据的更新非常困难。

- 工程师需要构建复杂的管道来读取整个分区或表，修改数据并将其写回。这种模式效率低，并且难以维护。

## Delta lake 架构

- 数据湖的典型架构是上层一个/或者多个分析引擎/或者其他计算框架，下层架设一个分布式存储系统，如下图左边所示。但是这种原始的数据湖用法是缺少管理的，比如缺少事务的支持，缺少数据质量的校验等等，一切数据管理完全靠人工手动保证。

![image-20201221212851424](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/21/21:28:51-image-20201221212851424.png)

- Delta Lake 就是在统一的存储层上面架上一层管理层，以解决人们手动管理数据湖数据的痛点。加上了一层管理层，首先我们就可以引入meta data管理，有了meta data管理，如果数据有schema，我们就可以管理schema，在数据入库的过程中对数据质量进行校验，并将不符合的数据剔除。
- 另外，管理了meta data，还可以实现ACIDTransactions，也就是事务的特性。在没有管理层的时候如果进行并发的操作，多个操作之间可能互相影响，比如一个用户在查询的时候另外一个用户进行了删除操作，有了事务的支持，就可以避免这种情况，在事务的支持下，每个操作都会生成一个快照，所有操作会生成一个快照序列，方便进行时间上的回溯，也就是时间旅行。
- Data Warehouse、Data Lake和Delta Lake三者的主要特性对比如下图所示。可以看出，Delta Lake相当于结合了Data Warehouse和Data Lake的优点，引入一个管理层，解决了大部分两者的缺点。

![image-20201221212909158](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/21/21:29:09-image-20201221212909158.png)

## Delta lake 特性

### 支持ACID事务

Delta Lake 在多并发写入之间提供 ACID 事务保证。每次写入都是一个事务，并且在事务日志中记录了写入的序列顺序。

事务日志跟踪文件级别的写入并使用乐观并发控制，这非常适合数据湖，因为多次写入/修改相同的文件很少发生。在存在冲突的情况下，Delta Lake 会抛出并发修改异常以便用户能够处理它们并重试其作业。

Delta Lake 还提供强大的可序列化隔离级别，允许工程师持续写入目录或表，并允许消费者继续从同一目录或表中读取。读者将看到阅读开始时存在的最新快照。

### Schema管理

Delta Lake 自动验证正在被写的 DataFrame 模式是否与表的模式兼容。

表中存在但 DataFrame 中不存在的列会被设置为 null 如果 DataFrame 中有额外的列在表中不存在，那么该操作将抛出异常 Delta Lake 具有可以显式添加新列的 DDL 和自动更新Schema 的能力 可伸缩的元数据处理

Delta Lake 将表或目录的元数据信息存储在事务日志中，而不是存储在元存储（metastore）中。这使得 Delta Lake 能够在固定的时间内列出大型目录中的文件，并且在读取数据时非常高效。

### 数据版本

Delta Lake 允许用户读取表或目录之前的快照。当文件被修改文件时，Delta Lake 会创建较新版本的文件并保留旧版本的文件。当用户想要读取旧版本的表或目录时，他们可以在 Apache Spark 的读取 API 中提供时间戳或版本号，Delta Lake 根据事务日志中的信息构建该时间戳或版本的完整快照。这允许用户重现之前的数据，并在需要时将表还原为旧版本的数据。

### 统一的批处理和流 sink

除了批处理写之外，Delta Lake 还可以使用作为 Apache Spark structured streaming 高效的流 sink。再结合 ACID 事务和可伸缩的元数据处理，高效的流 sink 现在支持许多接近实时的分析用例，而且无需维护复杂的流和批处理管道。

### 数据存储格式采用开源 Apache Parquet

Delta Lake 中的所有数据都是使用 Apache Parquet 格式存储，使 Delta Lake 能够利用 Parquet 原生的高效压缩和编码方案。

### 更新和删除

Delta Lake 支持 merge, update 和 delete 等 DML 命令。这使得数据工程师可以轻松地在数据湖中插入/更新和删除记录。 由于 Delta Lake 以文件级粒度跟踪和修改数据，因此它比读取和覆盖整个分区或表更有效。

### 数据异常处理

Delta Lake 还将支持新的 API 来设置表或目录的数据异常。工程师能够设置一个布尔条件并调整报警阈值以处理数据异常。当 Apache Spark 作业写入表或目录时，Delta Lake 将自动验证记录，当数据存在异常时，它将根据提供的设置来处理记录。

### 兼容 Apache Spark API

- 开发人员可以将 Delta Lake 与他们现有的数据管道一起使用，仅需要做一些细微的修改。

> Delta Lake 的源码大量使用了 Spark 的代码实现，而 Hudi 虽然也是 Spark 的一个类库，但是其读写底层文件时并不依赖 Spark 去实现，而是实现了自己独有的 InputFormat，比如 HoodieParquetInputFormat、HoodieParquetRealtimeInputFormat 等。所以从这个角度上看，Hudi 和其他系统去做兼容比 Delta Lake 要容易。

## 对比Hudi

### 数据 Merge 策略

- Delta Lake 支持对存储的数据进行更新，并且仅支持写入的时候进行数据合并（Write On Merge），它会获取需要更新的数据对应的文件，然后直接读取这些文件并使用 Spark 的 Join 进行计算产生新的文件。
- Hudi 也是支持写入数据的时候进行合并，但是相比 Delta Lake，Hudi 还支持 Read On Merge 模式，也就是将增量数据写入到一个 delta 文件，然后默认情况下在更新完数据后会启动一个作业进行 compaction 操作。当然，这个也是可以关闭的，也就是更新的时候值负责数据的写入，合并操作可以单独用一个作业来跑。
- 从功能上来说，**这方面 Hudi 比 Delta Lake 设计的要好**。在少写多读的情况下，Write On Merge 模式很不错；而在多写少读的情况下，Read On Merge 模式很不错，而 Delta Lake 目前还不支持 Read On Merge 模式。
- Hudi 提供了索引机制，在数据合并的时候，可以使用索引的信息快速定位到某行数据所在的文件，从而加快数据更新的速度。

### 和其他系统兼容性

- Delta Lake 的源码大量使用了 Spark 的代码实现，而 Hudi 虽然也是 Spark 的一个类库，但是其读写底层文件时并不依赖 Spark 去实现，而是实现了自己独有的 InputFormat，比如 HoodieParquetInputFormat、HoodieParquetRealtimeInputFormat 等。**所以从这个角度上看，Hudi 和其他系统去做兼容比 Delta Lake 要容易**。
- 目前 Hudi 原生支持 Spark、Presto、MapReduce 以及 Hive 等大数据生态系统，Flink 的支持正在开发中，参见 HUDI-184。
- Delta Lake 作为一个数据湖的存储层，其实不应该和Spark进行深入的绑定，数砖肯定意识到这个问题，所以数砖的大佬们弄了一个新的项目，用于和其他大数据生态系统进行整合，参见 https://github.com/delta-io/connectors，相关的 ISSUE 参见 https://github.com/delta-io/delta/issues/245。
- 在前几天发布的 Delta Lake 0.5.0 也支持 Hive、Presto 等常见的大数据查询引擎。

### SQL 语法

- SQL 的支持能够为用户提供极大的便利。从 0.4.0 版本开始，Delta Lake 已经开始支持一些命令的 SQL 语法了。由于 Delta Lake 是单独的一个项目，如果需要让它支持所有的 SQL 语法，需要从 Apache Spark 里面拷贝大量的代码到 Delta Lake 项目中，不便于维护，所以这个版本只支持 vacuum 和 history 简单命令的 SQL 语法。
- 其他的 delete、update 以及 merge 的 DML 操作支持可能得等到 Spark 3.0 版本才会支持的。目前社区也在 Spark 3.0 里面的 DataSource V2 API 里面添加了对 DELETE/UPDATE/MERGE 的支持
- Hudi 目前还不支持使用 SQL 进行 DDL / DML 相关操作，不过社区已经有小伙伴提到这个东西了

### 多语言支持

- Delta Lake 的多语言这块做的比较好，目前支持使用 Scala、Java 以及 Python（0.4.0 版本开始支持）；而 Apache Hudi 目前只支持 Scala 以及 Java 语言。

### 并发控制

- Delta Lake 原生提供了 ACID 事务的支持，每次写入都是一个事务，并且在事务日志中记录了写入的序列顺序。事务日志跟踪文件级别的写入并使用乐观并发控制。在存在冲突的情况下，Delta Lake 会抛出并发修改异常以便用户能够处理它们并重试其作业。Delta Lake 还提供强大的可序列化隔离级别，允许工程师持续写入目录或表，并允许消费者继续从同一目录或表中读取，读者将看到阅读开始时存在的最新快照。因为 ACID 的支持，Delta Lake 允许同一时刻多个用户同时写一张表。
- Apache Hudi 每次修改也会生成一个事务提交，但是其不支持同一时刻多个用户同时写一张表，因为如果是这种情况最后的数据是不对的。

### 版本回退

- Delta Lake 原生提供了一个叫做时间旅行（Time Travel）的功能，通过这个功能我们可以回退到任意版本的数据

```
val df = spark.read.format("delta").option("versionAsOf", 0).load("/tmp/delta-table")
df.show()
```

- 这个功能非常有用，比如我们发布新的系统时，然后发现了一个 Bug，这时候我们可以使用这个功能将数据回退到正确的状态。
- Apache Hudi 也支持类似的概念，也就是增量视图（Incremental View），我们可以通过指定一个时间段，然后查询到这个时间段新增或者修改的数据

```
//incrementally query data
val incViewDF = spark.read.format("org.apache.hudi").
    option(VIEW_TYPE_OPT_KEY, VIEW_TYPE_INCREMENTAL_OPT_VAL).
    option(BEGIN_INSTANTTIME_OPT_KEY, beginTime).
    option(END_INSTANTTIME_OPT_KEY, endTime).
    load(basePath);
incViewDF.registerTempTable("hudi_incr_table")
spark.sql("select `_hoodie_commit_time`, fare, begin_lon, begin_lat, ts from  hudi_incr_table where fare > 20.0").show()
```

### 小文件问题

- Databricks Runtime 里面的 Delta Lake 是支持使用 OPTIMIZE 命令将小文件进行压缩的
- Hudi 可以通过将对该分区中的插入作为对现有小文件的更新来解决小文件的问题,可以通过 hoodie.parquet.small.file.limit 参数实现

### 底层文件系统格式

- Delta Lake 中的所有数据都是使用 Apache Parquet 格式存储，使 Delta Lake 能够利用 Parquet 原生的高效压缩和编码方案。
- 在 Hudi 中，如果我们使用 Write On Merge 存储类型（默认），那么底层也是使用 Apache Parquet 格式存储数据；而如果使用了 Read On Merge存储类型，则同时使用Avro和Parquet来存储数据。其中Avro格式以行的形式存储增量的数据，而 Parquet 以列的形式存储已经 Merge 好的数据。

### 表模式管理

#### 新增列

- 在 Delta Lake 中，默认情况下，如果插入的数据包含 Delta Lake 表中没有的列,在写入数据的时候加上 .option("mergeSchema", "true") 参数即可，这时候 Delta Lake 表就会增加新的一列。

#### 缺省列处理

- 在 Delta Lake 中，如果我们插入的数据不包含对应表的全部字段，是可以插入进去的，并且缺少的列对应的数据为 null。但是在 Hudi 中，插入数据必须指定表的全部字段，否则会报错

#### 类型修改

- 目前 Delta Lake 和 Hudi 都不支持修改表中已有字段的类型。

### 总结

- Delta Lake 和 Hudi 两个都是数据湖的两款产品，各有优缺点。Hudi 的产生背景就是为了解决 Uber 的增量更新的问题，所以如果你对 Delta Lake 的数据 Merge 不是很满意，那么 Hudi 可能会适合你。**如果你对多客户端同时写一张表很在意，那么 Delta Lake 可能会更适合你**。