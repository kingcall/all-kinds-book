[TOC]

## 本地模式

0.7版本后Hive开始支持任务执行选择本地模式(local mode)大多数的Hadoop job是需要hadoop提供的完整的可扩展性来处理大数据的。

在hive中运行的sql有很多是比较小的sql,数据量小,计算量小 ，这些比较小的sql如果也采用分布式的方式来执行,那么是得不偿失的.因为sql真正执行的时间可能只有10秒,但是分布式任务的生成的其他过程的执行可能要1分钟.这样的小任务更适合采用lcoal mr的方式来执行.就是在本地来执行,通过把输入数据拉回客户端来执行.

### 启用方式

-  set hive.exec.mode.local.auto=true;(默认为false)

**当一个job满足如下条件才能真正使用本地模式**：

1. job的输入数据大小必须小于参数：hive.exec.mode.local.auto.inputbytes.max(默认128MB)
2. job的map数必须小于参数：hive.exec.mode.local.auto.tasks.max(默认4)
3. job的reduce数必须为0或者1

### 例子

我这里有一张表，其实就是我们在前面教程[hive streaming](https://blog.csdn.net/king14bhhb/article/details/111729038)中演示用到的,我先在不开启本地模式下执行一下

```sql
select weekday,count(1) from ods.u_data_new group by weekday;
```

下面是执行日志，我们看到大概花费了15.762 秒

```
Starting Job = job_1608438780277_0029, Tracking URL = http://localhost:8088/proxy/application_1608438780277_0029/
2020-12-27 12:51:01,739 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] exec.Task (SessionState.java:printInfo(1227)) - Starting Job = job_1608438780277_0029, Tracking URL = http://localhost:8088/proxy/application_1608438780277_0029/
Kill Command = /usr/local/Cellar/hadoop/3.2.1/libexec/bin/mapred job  -kill job_1608438780277_0029
2020-12-27 12:51:01,739 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] exec.Task (SessionState.java:printInfo(1227)) - Kill Command = /usr/local/Cellar/hadoop/3.2.1/libexec/bin/mapred job  -kill job_1608438780277_0029
Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2020-12-27 12:51:06,967 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] exec.Task (SessionState.java:printInfo(1227)) - Hadoop job information for Stage-1: number of mappers: 1; number of reducers: 1
2020-12-27 12:51:06,984 WARN  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] mapreduce.Counters (AbstractCounters.java:getGroup(235)) - Group org.apache.hadoop.mapred.Task$Counter is deprecated. Use org.apache.hadoop.mapreduce.TaskCounter instead
2020-12-27 12:51:06,984 Stage-1 map = 0%,  reduce = 0%
2020-12-27 12:51:06,984 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] exec.Task (SessionState.java:printInfo(1227)) - 2020-12-27 12:51:06,984 Stage-1 map = 0%,  reduce = 0%
2020-12-27 12:51:11,068 Stage-1 map = 100%,  reduce = 0%
2020-12-27 12:51:11,068 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] exec.Task (SessionState.java:printInfo(1227)) - 2020-12-27 12:51:11,068 Stage-1 map = 100%,  reduce = 0%
2020-12-27 12:51:16,171 Stage-1 map = 100%,  reduce = 100%
2020-12-27 12:51:16,172 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] exec.Task (SessionState.java:printInfo(1227)) - 2020-12-27 12:51:16,171 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_1608438780277_0029
2020-12-27 12:51:17,198 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] exec.Task (SessionState.java:printInfo(1227)) - Ended Job = job_1608438780277_0029
MapReduce Jobs Launched:
2020-12-27 12:51:17,206 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] ql.Driver (SessionState.java:printInfo(1227)) - MapReduce Jobs Launched:
Stage-Stage-1: Map: 1  Reduce: 1   HDFS Read: 1192486 HDFS Write: 227 SUCCESS
2020-12-27 12:51:17,206 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] ql.Driver (SessionState.java:printInfo(1227)) - Stage-Stage-1: Map: 1  Reduce: 1   HDFS Read: 1192486 HDFS Write: 227 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
2020-12-27 12:51:17,206 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] ql.Driver (SessionState.java:printInfo(1227)) - Total MapReduce CPU Time Spent: 0 msec
2020-12-27 12:51:17,206 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] ql.Driver (Driver.java:execute(2531)) - Completed executing command(queryId=liuwenqiang_20201227125101_07ce9455-4e6d-40f5-9e33-e9691a0a459d); Time taken: 15.655 seconds
OK
2020-12-27 12:51:17,206 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] ql.Driver (SessionState.java:printInfo(1227)) - OK
2020-12-27 12:51:17,206 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] ql.Driver (Driver.java:checkConcurrency(285)) - Concurrency mode is disabled, not creating a lock manager
2020-12-27 12:51:17,209 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] mapred.FileInputFormat (FileInputFormat.java:listStatus(259)) - Total input files to process : 1
2020-12-27 12:51:17,210 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] sasl.SaslDataTransferClient (SaslDataTransferClient.java:checkTrustAndSend(239)) - SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
2020-12-27 12:51:17,211 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] exec.ListSinkOperator (Operator.java:logStats(1038)) - RECORDS_OUT_OPERATOR_LIST_SINK_10:7, RECORDS_OUT_INTERMEDIATE:0,
1	12254
2	13579
3	14430
4	15114
5	14743
6	18229
7	11651
Time taken: 15.762 seconds, Fetched: 7 row(s)
```

现在我们开启本地执行模式再执行一下

```sql
 set hive.exec.mode.local.auto=true;
 select weekday,count(1) from ods.u_data_new group by weekday;
```

从下面的日志输出中我们看到了使用了`LocalJobRunner` 也就是启动了本地执行模式，最后的耗时只有 1.507秒，相比分布式的执行模式还是快了不少的，因为很多时候我们做测试啊或者本就数据量不大，这种情况下还是很有用的。

```
2020-12-27 12:55:05,589 INFO  [pool-18-thread-1] mapred.LocalJobRunner (LocalJobRunner.java:statusUpdate(634)) - reduce > reduce
2020-12-27 12:55:05,589 INFO  [pool-18-thread-1] mapred.Task (Task.java:sendDone(1380)) - Task 'attempt_local636843578_0002_r_000000_0' done.
2020-12-27 12:55:05,589 INFO  [pool-18-thread-1] mapred.Task (Task.java:done(1276)) - Final Counters for attempt_local636843578_0002_r_000000_0: Counters: 35
	File System Counters
		FILE: Number of bytes read=162523387
		FILE: Number of bytes written=83727359
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=2359027
		HDFS: Number of bytes written=81939664
		HDFS: Number of read operations=123
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=87
		HDFS: Number of bytes read erasure-coded=0
	Map-Reduce Framework
		Combine input records=0
		Combine output records=0
		Reduce input groups=7
		Reduce shuffle bytes=146
		Reduce input records=7
		Reduce output records=0
		Spilled Records=7
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=0
		Total committed heap usage (bytes)=252706816
	HIVE
		CREATED_FILES=1
		RECORDS_OUT_0=7
		RECORDS_OUT_INTERMEDIATE=0
		RECORDS_OUT_OPERATOR_FS_6=7
		RECORDS_OUT_OPERATOR_GBY_4=7
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Output Format Counters
		Bytes Written=0
2020-12-27 12:55:05,589 INFO  [pool-18-thread-1] mapred.LocalJobRunner (LocalJobRunner.java:run(353)) - Finishing task: attempt_local636843578_0002_r_000000_0
2020-12-27 12:55:05,589 INFO  [Thread-129] mapred.LocalJobRunner (LocalJobRunner.java:runTasks(486)) - reduce task executor complete.
2020-12-27 12:55:06,471 Stage-1 map = 100%,  reduce = 100%
2020-12-27 12:55:06,471 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] exec.Task (SessionState.java:printInfo(1227)) - 2020-12-27 12:55:06,471 Stage-1 map = 100%,  reduce = 100%
Ended Job = job_local636843578_0002
2020-12-27 12:55:06,472 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] exec.Task (SessionState.java:printInfo(1227)) - Ended Job = job_local636843578_0002
MapReduce Jobs Launched:
2020-12-27 12:55:06,477 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] ql.Driver (SessionState.java:printInfo(1227)) - MapReduce Jobs Launched:
Stage-Stage-1:  HDFS Read: 4718054 HDFS Write: 163879101 SUCCESS
2020-12-27 12:55:06,477 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] ql.Driver (SessionState.java:printInfo(1227)) - Stage-Stage-1:  HDFS Read: 4718054 HDFS Write: 163879101 SUCCESS
Total MapReduce CPU Time Spent: 0 msec
2020-12-27 12:55:06,478 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] ql.Driver (SessionState.java:printInfo(1227)) - Total MapReduce CPU Time Spent: 0 msec
2020-12-27 12:55:06,478 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] ql.Driver (Driver.java:execute(2531)) - Completed executing command(queryId=liuwenqiang_20201227125504_616a97cf-1b85-4a7f-aaf4-f565a7d3843b); Time taken: 1.346 seconds
OK
2020-12-27 12:55:06,478 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] ql.Driver (SessionState.java:printInfo(1227)) - OK
2020-12-27 12:55:06,478 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] ql.Driver (Driver.java:checkConcurrency(285)) - Concurrency mode is disabled, not creating a lock manager
2020-12-27 12:55:06,480 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] mapred.FileInputFormat (FileInputFormat.java:listStatus(259)) - Total input files to process : 1
2020-12-27 12:55:06,483 INFO  [680439f2-d65f-4c1f-8eaf-c5aee654b618 main] exec.ListSinkOperator (Operator.java:logStats(1038)) - RECORDS_OUT_OPERATOR_LIST_SINK_10:7, RECORDS_OUT_INTERMEDIATE:0,
1	12254
2	13579
3	14430
4	15114
5	14743
6	18229
7	11651
Time taken: 1.507 seconds, Fetched: 7 row(s)
```





## 严格模式

Hive提供了一个严格模式，可以防止用户执行那些可能产生意想不到的不好的效果的查询。说通俗一点就是这种模式可以阻止某些查询的执行。

### 启用方式

通过 `set hive.mapred.mode=strict;` 开启严格模式，严格模式不允许执行以下查询：

- 分区表上没有指定分区
- 没有limit限制的order by语句
- 笛卡尔积：JOIN时没有ON语句

### 例子

首先我们创建一个分区表，然后加载数据

```sql
CREATE TABLE ods.u_data (
  userid INT,
  movieid INT,
  rating INT,
  unixtime STRING)
 partitioned by(year string,month string ,day string) 
ROW FORMAT DELIMITED
FIELDS TERMINATED BY '\t'
STORED AS TEXTFILE;
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/ml-100k/u.data' OVERWRITE INTO TABLE ods.u_data partition(year='2020',month='2020-12',day='2020-12-21');
LOAD DATA LOCAL INPATH '/Users/liuwenqiang/ml-100k/u.data' OVERWRITE INTO TABLE ods.u_data partition(year='2020',month='2020-12',day='2020-12-22');
```

尝试执行没有添加分区条件的查询

```sql
select * from  ods.u_data;
```

我们可以看到报错了，提示说` Queries against partitioned tables without a partition filter are disabled for safety reasons`

```
Error 10056]: Queries against partitioned tables without a partition filter are disabled for safety reasons. If you know what you are doing, please set hive.strict.checks.no.partition.filter to false and make sure that hive.mapred.mode is not set to 'strict' to proceed. Note that you may get errors or incorrect results if you make a mistake while using some of the unsafe features. No partition predicate for Alias "u_data" Table "u_data"
org.apache.hadoop.hive.ql.parse.SemanticException: Queries against partitioned tables without a partition filter are disabled for safety reasons. If you know what you are doing, please set hive.strict.checks.no.partition.filter to false and make sure that hive.mapred.mode is not set to 'strict' to proceed. Note that you may get errors or incorrect results if you make a mistake while using some of the unsafe features. No partition predicate for Alias "u_data" Table "u_data"
	at org.apache.hadoop.hive.ql.optimizer.ppr.PartitionPruner.prune(PartitionPruner.java:192)
	at org.apache.hadoop.hive.ql.optimizer.ppr.PartitionPruner.prune(PartitionPruner.java:147)
	at org.apache.hadoop.hive.ql.parse.ParseContext.getPrunedPartitions(ParseContext.java:532)
	at org.apache.hadoop.hive.ql.optimizer.SimpleFetchOptimizer.checkTree(SimpleFetchOptimizer.java:211)
	at org.apache.hadoop.hive.ql.optimizer.SimpleFetchOptimizer.optimize(SimpleFetchOptimizer.java:144)
	at org.apache.hadoop.hive.ql.optimizer.SimpleFetchOptimizer.transform(SimpleFetchOptimizer.java:114)
	at org.apache.hadoop.hive.ql.optimizer.Optimizer.optimize(Optimizer.java:250)
	at org.apache.hadoop.hive.ql.parse.SemanticAnalyzer.analyzeInternal(SemanticAnalyzer.java:12295)
	at org.apache.hadoop.hive.ql.parse.CalcitePlanner.analyzeInternal(CalcitePlanner.java:330)
	at org.apache.hadoop.hive.ql.parse.BaseSemanticAnalyzer.analyze(BaseSemanticAnalyzer.java:285)
	at org.apache.hadoop.hive.ql.Driver.compile(Driver.java:659)
	at org.apache.hadoop.hive.ql.Driver.compileInternal(Driver.java:1826)
	at org.apache.hadoop.hive.ql.Driver.compileAndRespond(Driver.java:1773)
	at org.apache.hadoop.hive.ql.Driver.compileAndRespond(Driver.java:1768)
	at org.apache.hadoop.hive.ql.reexec.ReExecDriver.compileAndRespond(ReExecDriver.java:126)
	at org.apache.hadoop.hive.ql.reexec.ReExecDriver.run(ReExecDriver.java:214)
	at org.apache.hadoop.hive.cli.CliDriver.processLocalCmd(CliDriver.java:239)
	at org.apache.hadoop.hive.cli.CliDriver.processCmd(CliDriver.java:188)
	at org.apache.hadoop.hive.cli.CliDriver.processLine(CliDriver.java:402)
	at org.apache.hadoop.hive.cli.CliDriver.executeDriver(CliDriver.java:821)
	at org.apache.hadoop.hive.cli.CliDriver.run(CliDriver.java:759)
	at org.apache.hadoop.hive.cli.CliDriver.main(CliDriver.java:683)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.hadoop.util.RunJar.run(RunJar.java:323)
	at org.apache.hadoop.util.RunJar.main(RunJar.java:236)
```

