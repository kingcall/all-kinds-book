[TOC]

## Usage

### Configuration Variables

有关配置 Hive 表统计信息的变量列表，请参见[Configuration Properties](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/Configuration_Properties.html)中的[Statistics](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/Configuration_Properties.html#ConfigurationProperties-Statistics)。 [Configuring Hive](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/AdminManual_Configuration.html#AdminManualConfiguration-ConfiguringHive)描述了如何使用变量。

### 新创建的表

对于新创建的表和/或分区(通过[INSERT OVERWRITE](http://wiki.apache.org/hadoop/Hive/LanguageManual/DML#Inserting_data_into_Hive_Tables_from_queries)命令填充)，默认情况下会自动计算统计信息。用户必须将布尔变量 **[hive.stats.autogather](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/Configuration_Properties.html#ConfigurationProperties-hive.stats.autogather)** 显式设置为 **false** ，以便不会自动计算统计信息并将其存储到 Hive MetaStore 中。

```shell
set hive.stats.autogather=false;
```

用户还可以设置变量 **[hive.stats.dbclass](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/Configuration_Properties.html#ConfigurationProperties-hive.stats.dbclass)** 来指定用于存储临时统计信息的实现。例如，要将 HBase 设置为临时统计信息存储的实现(默认为`jdbc:derby`或`fs`，具体取决于 Hive 版本)，用户应发出以下命令：

```shell
set hive.stats.dbclass=hbase;
```

如果是临时存储统计信息的 JDBC 实现(例如 Derby 或 MySQL)，则用户应通过设置变量 **[hive.stats.dbconnectionstring](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/Configuration_Properties.html#ConfigurationProperties-hive.stats.dbconnectionstring)** 指定与数据库的适当连接字符串。用户还应该通过设置变量 **[hive.stats.jdbcdriver](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/Configuration_Properties.html#ConfigurationProperties-hive.stats.jdbcdriver)** 来指定适当的 JDBC 驱动程序。

```shell
set hive.stats.dbclass=jdbc:derby;
set hive.stats.dbconnectionstring="jdbc:derby:;databaseName=TempStatsStore;create=true";
set hive.stats.jdbcdriver="org.apache.derby.jdbc.EmbeddedDriver";
```

查询可能无法完全准确地收集统计信息。如果无法可靠地收集统计信息，则设置 **[hive.stats.reliable](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/Configuration_Properties.html#ConfigurationProperties-hive.stats.reliable)** 将使查询失败。默认情况下是`false`。

### 现有表格–分析

对于现有表和/或分区，用户可以发出 ANALYZE 命令来收集统计信息并将其写入 Hive MetaStore。该命令的语法如下所述：

```shell
ANALYZE TABLE [db_name.]tablename [PARTITION(partcol1[=val1], partcol2[=val2], ...)]  -- (Note: Fully support qualified table name since Hive 1.2.0, see HIVE-10007.)
  COMPUTE STATISTICS
  [FOR COLUMNS]          -- (Note: Hive 0.10.0 and later.)
  [CACHE METADATA]       -- (Note: Hive 2.1.0 and later.)
  [NOSCAN];
```

当用户发出该命令时，他可能会或可能不会指定分区规范。如果用户未指定任何分区规范，则将收集表以及所有分区(如果有)的统计信息。如果指定了某些分区规范，则仅收集那些分区的统计信息。在跨所有分区计算统计信息时，仍需要列出分区列。从[Hive 1.2.0](https://issues.apache.org/jira/browse/HIVE-10007)开始，Hive 在此命令中完全支持限定的表名。如果使用了非限定的表名，则用户只能计算当前数据库下表的统计信息。

当指定了可选参数 NOSCAN 时，该命令将不会扫描文件，因此应该是快速的。代替所有统计信息，它仅收集以下统计信息：

- 文件数
- 物理大小(以字节为单位)

Version 0.10.0: FOR COLUMNS

从[Hive 0.10.0](https://issues.apache.org/jira/browse/HIVE-1362)开始，可选参数 FOR COLUMNS 为指定表中的所有列(如果表已分区，则为所有分区)计算列统计信息。有关详情，请参见[Hive 中的列统计](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/Column_Statistics_in_Hive.html)。

要显示这些统计信息，请使用 DESCRIBE FORMATTED [* db_name *。] * table_name * * column_name * [PARTITION(* partition_spec *)]。

## Examples

假设表 Table1 具有 4 个分区，其规格如下：

- 分区 1：(ds ='2008-04-08'，hr = 11)
- 分区 2：(ds ='2008-04-08'，hr = 12)
- 分区 3：(ds ='2008-04-09'，hr = 11)
- 分区 4：(ds ='2008-04-09'，hr = 12)

然后发出以下命令：

```shell
ANALYZE TABLE Table1 PARTITION(ds='2008-04-09', hr=11) COMPUTE STATISTICS;
```

然后仅收集分区 3(ds ='2008-04-09'，hr = 11)的统计信息。

如果发出命令：

```shell
ANALYZE TABLE Table1 PARTITION(ds='2008-04-09', hr=11) COMPUTE STATISTICS FOR COLUMNS;
```

然后收集分区 3 的所有列的列统计信息(ds ='2008-04-09'，hr = 11)。在 Hive 0.10.0 和更高版本中可用。

如果发出命令：

```shell
ANALYZE TABLE Table1 PARTITION(ds='2008-04-09', hr) COMPUTE STATISTICS;
```

然后仅收集分区 3 和分区 4 的统计信息(hr = 11 和 hr = 12)。

如果发出命令：

```shell
ANALYZE TABLE Table1 PARTITION(ds='2008-04-09', hr) COMPUTE STATISTICS FOR COLUMNS;
```

然后仅收集分区 3 和分区 4(Hive 0.10.0 及更高版本)的所有列的列统计信息。

如果发出命令：

```shell
ANALYZE TABLE Table1 PARTITION(ds, hr) COMPUTE STATISTICS;
```

然后收集所有四个分区的统计信息。

如果发出命令：

```shell
ANALYZE TABLE Table1 PARTITION(ds, hr) COMPUTE STATISTICS FOR COLUMNS;
```

然后收集所有四个分区(Hive 0.10.0 及更高版本)的所有列的列统计信息。

对于非分区表，可以发出以下命令：

```shell
ANALYZE TABLE Table1 COMPUTE STATISTICS;
```

收集表格的统计信息。

对于非分区表，可以发出以下命令：

```shell
ANALYZE TABLE Table1 COMPUTE STATISTICS FOR COLUMNS;
```

收集表的列统计信息(Hive 0.10.0 及更高版本)。

如果 Table1 是分区表，则对于基本统计信息，您必须像上面的 analyst 语句中那样指定分区规范。否则，将引发语义分析器异常。

但是，对于列统计信息，如果在 analyze 语句中未给出分区规范，则将计算所有分区的统计信息。

您可以通过发出[DESCRIBE](http://wiki.apache.org/hadoop/Hive/LanguageManual/DDL?highlight=(describe)#Describe_Partition)命令来查看存储的统计信息。统计信息存储在 Parameters 数组中。假设您对整个表 Table1 发出了 analyze 命令，然后发出了该命令：

```shell
DESCRIBE EXTENDED TABLE1;
```

然后在输出中，将显示以下内容：

```shell
... , parameters:{numPartitions=4, numFiles=16, numRows=2000, totalSize=16384, ...}, ....
```

如果发出命令：

```shell
DESCRIBE EXTENDED TABLE1 PARTITION(ds='2008-04-09', hr=11);
```

然后在输出中，将显示以下内容：

```shell
... , parameters:{numFiles=4, numRows=500, totalSize=4096, ...}, ....
```

如果发出命令：

```shell
desc formatted concurrent_delete_different partition(ds='tomorrow') name;
```

输出如下所示：

```shell
+-----------------+--------------------+-------+-------+------------+-----------------+--------------+--------------+------------+-------------+------------+----------+
|    col_name     |     data_type      |  min  |  max  | num_nulls  | distinct_count  | avg_col_len  | max_col_len  | num_trues  | num_falses  | bitvector  | comment  |
+-----------------+--------------------+-------+-------+------------+-----------------+--------------+--------------+------------+-------------+------------+----------+
| col_name        | name               | NULL  | NULL  | NULL       | NULL            | NULL         | NULL         | NULL       | NULL        | NULL       | NULL     |
| data_type       | varchar(50)        | NULL  | NULL  | NULL       | NULL            | NULL         | NULL         | NULL       | NULL        | NULL       | NULL     |
| min             |                    | NULL  | NULL  | NULL       | NULL            | NULL         | NULL         | NULL       | NULL        | NULL       | NULL     |
| max             |                    | NULL  | NULL  | NULL       | NULL            | NULL         | NULL         | NULL       | NULL        | NULL       | NULL     |
| num_nulls       | 0                  | NULL  | NULL  | NULL       | NULL            | NULL         | NULL         | NULL       | NULL        | NULL       | NULL     |
| distinct_count  | 2                  | NULL  | NULL  | NULL       | NULL            | NULL         | NULL         | NULL       | NULL        | NULL       | NULL     |
| avg_col_len     | 5.0                | NULL  | NULL  | NULL       | NULL            | NULL         | NULL         | NULL       | NULL        | NULL       | NULL     |
| max_col_len     | 5                  | NULL  | NULL  | NULL       | NULL            | NULL         | NULL         | NULL       | NULL        | NULL       | NULL     |
| num_trues       |                    | NULL  | NULL  | NULL       | NULL            | NULL         | NULL         | NULL       | NULL        | NULL       | NULL     |
| num_falses      |                    | NULL  | NULL  | NULL       | NULL            | NULL         | NULL         | NULL       | NULL        | NULL       | NULL     |
| bitVector       |                    | NULL  | NULL  | NULL       | NULL            | NULL         | NULL         | NULL       | NULL        | NULL       | NULL     |
| comment         | from deserializer  | NULL  | NULL  | NULL       | NULL            | NULL         | NULL         | NULL       | NULL        | NULL       | NULL     |
+-----------------+--------------------+-------+-------+------------+-----------------+--------------+--------------+------------+-------------+------------+----------+
```

如果发出命令：

```shell
ANALYZE TABLE Table1 PARTITION(ds='2008-04-09', hr) COMPUTE STATISTICS NOSCAN;
```

然后仅收集分区 3 和 4 的统计信息，文件数和以字节为单位的物理大小。

### {#StatsDev-ANALYZETABLE<table1>CACHEMETADATA} ANALYZE TABLE<table1> CACHE METADATA

Feature not implemented

HBase 上的 Hive Metastore 已停产，并在 Hive 3.0.0 中删除。见[HBaseMetastoreDevelopmentGuide](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/HBaseMetastoreDevelopmentGuide.html)

将 Hive Metastore 配置为使用 HBase 时，此命令在 HBase Metastore 中显式缓存文件元数据。

*此功能的目的是缓存文件元数据(例如 ORC 文件页脚)，以避免在拆分生成时从 HDFS 读取大量文件，并潜在地缓存一些有关拆分的信息(例如，根据位置分组有利于一段时间)以进一步加快生成速度，并通过一致的分割来实现更好的缓存位置.*

```shell
ANALYZE TABLE Table1 CACHE METADATA;
```

查看[HBase Metastore 拆分缓存](https://issues.apache.org/jira/secure/attachment/12749746/HBase metastore split cache.pdf)和([HIVE-12075](https://issues.apache.org/jira/browse/HIVE-12075))中的功能详细信息

## 当前状态(JIRA)

统计信息(例如表或分区的行数以及特定关注列的直方图)在许多方面都很重要。统计信息的关键用例之一是查询优化。统计信息是优化器成本函数的 Importing，因此它可以比较不同的计划并从中进行选择。统计信息有时可能满足用户查询的目的。用户只需查询存储的统计信息，而无需执行长期运行的执行计划，即可快速获得某些查询的答案。例如，获取用户年龄分布的分位数，人们使用的前 10 个应用程序以及不同会话的数量。

### 表和分区统计信息

支持统计信息的第一个里程碑是支持表和分区级别的统计信息。现在，表和分区统计信息存储在 Hive Metastore 中，用于新建或现有表。分区当前支持以下统计信息：

- 行数
- 文件数
- 大小(以字节为单位)

对于表，增加表的分区数量即可支持相同的统计信息。

Version: Table and partition statistics

表和分区级别统计信息由[HIVE-1361](https://issues.apache.org/jira/browse/HIVE-1361)添加到 Hive 0.7.0 中。



### Column Statistics

第二个里程碑是支持列级统计。请参阅设计文档中的[Hive 中的列统计](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/Column_Statistics_in_Hive.html)。

Version: Column statistics

列级统计信息已由[HIVE-1362](https://issues.apache.org/jira/browse/HIVE-1362)添加到 Hive 0.10.0 中。

### 前 K 个统计信息

[列级别的前 K 个统计信息](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/Top_K_Stats.html)仍在 await 中；参见[HIVE-3421](https://issues.apache.org/jira/browse/HIVE-3421)。

## Quick overview

| Description                                                  | Stored in                                                   | Collected by                                                 | Since                                                        |
| ------------------------------------------------------------ | ----------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 数据集组成的分区数                                           | 虚构的 metastore 属性： **numPartitions**                   | 在显示分区表的属性期间计算                                   | [Hive 2.3](https://issues.apache.org/jira/browse/HIVE-16315) |
| 数据集包含的文件数                                           | Metastore 表属性： **numFiles**                             | 在 Metastore 操作期间自动                                    |                                                              |
| 在文件系统级别看到的数据集的总大小                           | Metastore 表属性： **totalSize** ****                       |                                                              |                                                              |
| 数据集的未压缩大小                                           | Metastore 表属性： **rawDataSize**                          | 经计算，这些是基本统计数据。启用[hive.stats.autogather](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/Configuration_Properties.html#ConfigurationProperties-hive.stats.autogather)时自动计算。 |                                                              |
| 可以通过以下方式手动收集：ANALYZE TABLE ... COMPUTE STATISTICS | [Hive 0.8](https://issues.apache.org/jira/browse/HIVE-2185) |                                                              |                                                              |
| 数据集包含的行数                                             | 元存储表属性： **numRows**                                  |                                                              |                                                              |
| 列级统计信息                                                 | Metastore； TAB_COL_STATS 表                                | 计算，启用[hive.stats.column.autogather](https://www.docs4dev.com/docs/zh/apache-hive/3.1.1/reference/Configuration_Properties.html#ConfigurationProperties-hive.stats.column.autogather)时自动计算。 可以通过以下方式手动收集：分析表...列的计算统计信息 |                                                              |



## Implementation

对于新创建的表和现有的表，统计信息的计算方式相似。

对于新创建的表，创建新表的作业是 MapReduce 作业。在创建期间，每个 Map 器在 FileSink 运算符中从源表复制行时，都会收集遇到的行的统计信息并将其发布到数据库中(可能是[MySQL](http://www.mysql.com/))。在 MapReduce 作业结束时，已发布的统计信息将被汇总并存储在 MetaStore 中。

对于已经存在的表，将发生类似的过程，其中创建仅 Map 的作业，并且在 TableScan 运算符中处理表时，每个 Map 器都将收集遇到的行的统计信息，然后 continue 相同的过程。

显然，需要一种存储临时收集的统计信息的数据库。当前有两种实现，一种使用[MySQL](http://www.mysql.com/)，另一种使用[HBase](http://wiki.apache.org/hadoop/Hbase)。开发人员可以实现两个可插拔接口 IStatsPublisher 和 IStatsAggregator 来支持任何其他存储。接口在下面列出：

```shell
package org.apache.hadoop.hive.ql.stats;

import org.apache.hadoop.conf.Configuration;

/**
 * An interface for any possible implementation for publishing statics.
 */

public interface IStatsPublisher {

  /**
 * This method does the necessary initializations according to the implementation requirements.
   */
  public boolean init(Configuration hconf);

  /**
 * This method publishes a given statistic into a disk storage, possibly HBase or MySQL.
   *
 * rowID : a string identification the statistics to be published then gathered, possibly the table name + the partition specs.
   *
 * key : a string noting the key to be published. Ex: "numRows".
   *
 * value : an integer noting the value of the published key.
 * */
  public boolean publishStat(String rowID, String key, String value);

  /**
 * This method executes the necessary termination procedures, possibly closing all database connections.
   */
  public boolean terminate();

}
package org.apache.hadoop.hive.ql.stats;

import org.apache.hadoop.conf.Configuration;

/**
 * An interface for any possible implementation for gathering statistics.
 */

public interface IStatsAggregator {

  /**
 * This method does the necessary initializations according to the implementation requirements.
   */
  public boolean init(Configuration hconf);

  /**
 * This method aggregates a given statistic from a disk storage.
 * After aggregation, this method does cleaning by removing all records from the disk storage that have the same given rowID.
   *
 * rowID : a string identification the statistic to be gathered, possibly the table name + the partition specs.
   *
 * key : a string noting the key to be gathered. Ex: "numRows".
   *
 * */
  public String aggregateStats(String rowID, String key);

  /**
 * This method executes the necessary termination procedures, possibly closing all database connections.
   */
  public boolean terminate();

}
```