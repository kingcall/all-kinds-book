# 归档以减少文件计数

注意：由于涉及到警告，应将归档视为高级命令。

## Overview

由于 HDFS 的设计，文件系统中文件的数量直接影响 namenode 中的内存消耗。虽然通常对于小型群集而言不是问题，但是当文件数大于 50 到 1 亿时，内存使用量可能会达到一台计算机上可访问内存的限制。在这种情况下，具有尽可能少的文件是有利的。

使用[Hadoop Archives](http://hadoop.apache.org/docs/stable1/hadoop_archives.html)是减少分区中文件数量的一种方法。 Hive 具有内置支持，可将现有分区中的文件转换为 Hadoop 存档(HAR)，这样一个曾经由 100 个文件组成的分区只能占用约 3 个文件(取决于设置)。然而，权衡是由于从 HAR 读取时的额外开销，查询可能会变慢。

请注意，存档不会 zipfile-HAR 与 Unix tar 命令类似。

## Settings

使用归档之前，应配置 3 个设置。 (显示示例值.)

```shell
hive> set hive.archive.enabled=true;
hive> set hive.archive.har.parentdir.settable=true;
hive> set har.partfile.size=1099511627776;
```

`hive.archive.enabled`控制是否启用存档操作。

`hive.archive.har.parentdir.settable`通知 Hive 在创建存档时是否可以设置父目录。在最新版本的 Hadoop 中，`-p`选项可以指定存档的根目录。例如，如果以`/dir1`作为父目录归档`/dir1/dir2/file`，则生成的归档文件将包含目录结构`dir2/file`。在较早版本的 Hadoop(2011 年之前)中，此选项不可用，因此必须配置 Hive 来适应此限制。

`har.partfile.size`控制组成 Files 的文件的大小。Files 将包含`size_of_partition` `/` `har.partfile.size`个文件，四舍五入。较高的值表示较少的文件，但由于 Map 器数量减少，将导致较长的归档时间。

## Usage

### Archive

设置配置值后，可以使用以下命令将分区归档：

```shell
ALTER TABLE table_name ARCHIVE PARTITION (partition_col = partition_col_value, partition_col = partiton_col_value, ...)
```

For example:

```shell
ALTER TABLE srcpart ARCHIVE PARTITION(ds='2008-04-08', hr='12')
```

发出命令后，mapreduce 作业将执行归档。与 Hive 查询不同，CLI 上没有输出来指示进程。

### Unarchive

可以使用 unarchive 命令将分区恢复为原始文件：

```shell
ALTER TABLE srcpart UNARCHIVE PARTITION(ds='2008-04-08', hr='12')
```

## 注意和限制

- 在某些较旧的 Hadoop 版本中，HAR 有一些错误，可能会导致数据丢失或其他错误。确保将这些补丁集成到您的 Hadoop 版本中：

https://issues.apache.org/jira/browse/HADOOP-6591(已在 Hadoop 0.21.0 中修复)

https://issues.apache.org/jira/browse/MAPREDUCE-1548(已在 Hadoop 0.22.0 中修复)

https://issues.apache.org/jira/browse/MAPREDUCE-2143(已在 Hadoop 0.22.0 中修复)

https://issues.apache.org/jira/browse/MAPREDUCE-1752(已在 Hadoop 0.23.0 中修复)

- HarFileSystem 类仍然有一个尚待修复的错误：

https://issues.apache.org/jira/browse/MAPREDUCE-1877(2014 年移至https://issues.apache.org/jira/browse/HADOOP-10906)

Hive 带有 HiveHarFileSystem 类，该类解决了其中的一些问题，默认情况下是`fs.har.impl`的值。如果要滚动自己的 HarFileSystem 版本，请记住以下几点：

- 默认的 HiveHarFileSystem.getFileBlockLocations()具有 **no locality** 。这意味着它可能会引入更高的网络负载或降低性能。
- 无法使用 INSERT OVERWRITE 覆盖已归档的分区。分区必须先取消存档。
- 如果两个进程尝试同时归档同一分区，则可能会发生不良情况。 (需要实现并发支持.)



在内部，对分区进行存档时，将使用该分区原始位置(例如`/warehouse/table/ds=1`)中的文件创建一个 HAR。分区的父目录被指定为与原始位置相同，并且生成的归档文件名为“ data.har”。存档已移动到原始目录(例如`/warehouse/table/ds=1/data.har`)下，并且分区的位置已更改为指向存档。