[TOC]

## 事务

### 1. 背景

在我们使用的hive中一般它是不会支持事务的，因为hive的存储基于HDFS，HDFS 的文件，只能允许新建，删除，对文件中的内容进行更新，不允许单条修改，这也就是hive 不支持update和delete功能的原因

Hive 开始支持事务，是在 Hive 0.14 之后，如果一个表要实现update和delete功能，该表就必须支持ACID，而支持ACID，就必须满足以下条件：

1、表的存储格式必须是ORC（STORED AS ORC）；

3、Table property中参数transactional必须设定为True（tblproperties('transactional'='true')）；

4、以下配置项必须被设定：你也可以将这些参数配置到hive-site.xml 中去，但是不建议这样做

Client端：
```sql
set hive.support.concurrency=true
set hive.enforce.bucketing=true --（从Hive2.0不再需要）
set hive.exec.dynamic.partition.mode=nonstrict
set hive.txn.manager=org.apache.hadoop.hive.ql.lockmgr.DbTxnManager
```
服务端：
```sql
set hive.compactor.initiator.on=true
set hive.compactor.worker.threads=1
```



###  2. 什么是ACID，为什么要使用ACID呢？

ACID代表了数据库事务的四个特征：原子性(一个操作要么完全成功，要么失败，绝不会留下部分数据) 、一致性(一旦应用程序执行了一个操作，该操作的结果在它以后的每个操作中都是可见的)、隔离性(一个用户未完成的操作不会对其他用户造成意外影响)，以及持久性(一旦一个操作完成，它将保持下来，即便面对机器故障或系统故障)。一直以来，这些特性被认为是数据库系统事务功能的一部分。

截止到 0.13，Hive都只提供分区级别的原子性、一致性和持久性。可以通过打开一种可用的锁机制(Zookeeper或内存)来提供隔离性。随着在Hive 0.13中添加事务，现在可以在行级别提供完整的ACID语义，这样一个应用程序就可以在另一个应用程序读取同一个分区时添加行，而不会相互干扰。

Hive中添加ACID语义事务的特性，可以用于以下场景：

1.  流数据的采集。许多用户使用如Apache Flume，Apache Storm，或Apache Kafka等工具，将数据流到自己的Hadoop集群。尽管这些工具可以每秒数百行或更多行的速度写入数据，但Hive只能每十五分钟到一小时的添加分区。更短间隔的添加分区会导致表中出现大量的分区。这些工具可以将数据流到现有的分区中，但这将导致读操作产生脏读(也就是说，能看到在启动查询后会写入的数据)，并在目录中留下许多小文件，这将给NameNode带来压力。使用此事务功能，将支持此场景，同时保证读操作获得一致的数据视图，并避免过多的文件。
2.  缓慢变化的维度（表）。在典型的星型模型的数据仓库中，维度表随着时间的推移变化很缓慢。例如，零售商开设新的商店，这些商店需要添加到商店表中，或者现有的商店可能会改变面积或其他可跟踪的特性。这些更改会导致插入单个记录或更新记录(取决于所选择的策略)。从0.14开始，Hive能够支持这一点。
3.  数据更新 从Hive0.14开始，这些用例可以通过INSERT，UPDATE和DELETE来支持。
4.  使用SQL MERGE语句实现批量更新。

### 3 限制

1. 所有DML操作都是自动提交，尚不支持BEGIN，COMMIT，和ROLLBACK，计划在将来的版本中支持这些特性。
2. 在第一个版本中只支持ORC文件格式。构建事务特性的初衷是可以使用任何存储格式，只要这些存储格式可以确定如何更新或删除基本记录(基本上，具有显式或隐式的行id即可)，但到目前为止，仅完成针对ORC的集成工作。
3. 默认情况下，事务被配置为OFF。有关配置值的讨论，请参见下面的配置部分。
5. 不允许从非ACID的会话中读取/写入ACID表。换句话说，要操作ACID表，必须将Hive事务管理器设置为org.apache.hadoop.hive.ql.lockmgr.DbTxnManager。
6. 现在只支持快照隔离级别。当一个给定的查询启动时，它会提供该数据的一致快照。不支持脏读（READ UNCIMMITTED）、提交读（READ COMMITTED）、可重复读（REPEATABLE READ）或可序列化（SERIALIZABLE）。引入BEGIN的目的是在事务持续时间内支持快照隔离，而不仅仅是一个查询。根据用户请求，还可以添加其他隔离级别。
7. 现有的ZooKeeper和内存锁管理器与事务不兼容。我们无意处理这个问题。有关如何为事务存储锁的讨论，请参阅下面的基本设计。
8. 原来ACID表不支持使用ALTER TABLE更改Schema，参见Hive-11421。此缺陷于Hive 1.3.0/ Hive 2.0.0修复成功。
9. 使用Oracle作为Metastore DB和“datanucleus.connectionPoolingType = BONECP”时，可能偶发性产生 “No such lock ...”和“No such transaction...”的错误。在这种情况下，建议设置“datanucleus.connectionPoolingType = DBCP”。
10. LOAD DATA ...语句不支持事务性表，直到Hive-16732才能正确执行。



### 4 语法变化

1. 从Hive0.14开始,INSERT … VALUES，UPDATE和DELETE已被添加到SQL语法。详见 [Language Manual DML](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DML)。
2. 2.一些新的命令已经添加到Hive的DDL中，以支持ACID和事务，另外一些修改了现有的DDL语句。
3. 加入一个新命令SHOW TRANSACTIONS，详见[Show Transactions](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ShowTransactions)。
4. 加入一个新命令SHOW COMPACTIONS，详见 [Show Compactions](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ShowCompactions)。
5. 改变了SHOW LOCKS命令，以便提供锁与事务相关的信息。如果您使用的是Zookeeper或内存锁管理器，你会发现这个命令的输出并无差别。详见[Show Locks](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ShowLocks)。
6. ALTER TABLE添加了一个新的选项，用于表或分区的合并。在一般情况下，用户不需要请求合并，因为系统会检测合并的需求并开始执行合并。但是，如果关闭了表的合并功能，或者用户希望在系统没选择的情况下合并表，则可以使用ALTER TABLE启动合并。详见 [Alter Table/Partition合并](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AlterTable/PartitionCompact)，这将任务放到合并排队等待后返回。用户可以使用SHOW COMPACTIONS查看合并的进度。
7. 添加一个新命令终止事务：ABORT TRANSACTIONS，详见[Abort Transactions](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-AbortTransactions)。

### 5. 实现原理

Hive 的文件存储是基于 HDFS 文件存在的，所以原则上不会直接对 HDFS 做文件内容的事务更新，只能是采取另外的手段来完成。

即用 HDFS 文件作为原始数据，用 delta 文件作为操作日志的记录，也就是说新记录、更新和删除的记录存储在增量文件中。当访问 Hive 数据时，根据 HDFS 文件和 delta 文件做合并，查询最新的数据。每个事务(或者是流代理(如Flume或Storm)的每个批事务)都创建了一组新的增量文件，以更改表或分区。在读取时，合并基础文件和增量文件并应用更新和删除的变化。

#### 5.1 演示

下面我么创建一个事务表，并插入几条数据看看

```sql
CREATE TABLE ods.employee (
    id int, name string, salary int)
ROW FORMAT DELIMITED
    FIELDS TERMINATED BY '\t'
    STORED AS ORC TBLPROPERTIES ('transactional' = 'true');
INSERT INTO employee VALUES(1, 'kingcall', 10000);
INSERT INTO employee VALUES(1, 'kingcall', 10000);
INSERT INTO employee VALUES(1, 'kingcall', 10000);
```

然后我们去HDFS 上看看

![image-20201228103916837](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/10:39:17-image-20201228103916837.png)

正在执行中的事务，是以一个`satging`开头的文件夹维护的，执行结束就是delta 文件夹了，每次执行一次事务操作都会有这样的一个delta增量文件夹

接下来我们执行一个update 操作,给员工调整一下工资

```
UPDATE employee SET salary=100000 WHERE id=1;
```

![image-20201228105832234](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/10:58:33-image-20201228105832234.png)

可以看到 所有 INSERT 语句都会创建 `delta` 目录。UPDATE 语句也会创建 `delta` 目录，但会先创建一个 `delete` 目录，即先删除、后插入。`delete` 目录的前缀是 delete_delta；

每个事务的文件夹，都有两个文件，一个是数据文件，一个事务的信息文件

![image-20201228110537339](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/11:05:37-image-20201228110537339.png)

关于数据信息你可以直接在hive 中查看，关于事务的信息可以在hive中 通过 `row__id` 这个虚拟列进行查看，当然你也可以通过ORC 文件的工具进行查看

![image-20201228111527388](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/11:15:28-image-20201228111527388.png)

####  5.2 合并器(Compactor)



合并器是一套Metastore内运行，支持ACID系统的后台进程。它由Initiator（发起者），Worker，Cleaner，AcidHouseKeeperService和其他几部分组成。

##### 增量文件合并

![image-20201228104500712](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/28/10:45:01-image-20201228104500712.png)

随着表的修改操作，创建了越来越多的增量文件，可以看上图，就需要合并以保持足够的性能。有两种类型的合并，次要和主要：

	1. Minor Compaction会将所有的 `delta` 文件压缩为一个文件，`delete` 也压缩为一个。压缩后的结果文件名中会包含写事务 ID 范围 `ALTER TABLE employee COMPACT 'minor';`
 	2. Major Compaction 则会将所有文件合并为一个文件。主要合并成本更高，但更有效。

所有合并都是在后台完成的，不会阻止数据的并发读、写。合并后，系统将等待所有旧文件的读操作完成后，删除旧文件。

#####  Initiator（发起者）

这个模块负责发现哪些表或分区需要合并。需要在Metastore中配置参数hive.compactor.initiator.on启用该模块，在“事务的新配置参数”中有几个形式为*.threshold的属性，控制何时创建合并任务以及执行哪种类型的合并。每个合并任务处理一个分区(如果表未分区，则处理整个表)。如果给定分区的连续合并失败次数超过hive.compactor.initiator.failed.compacts.threshold，则该分区的自动合并调度将停止。有关更多信息，请参见配置参数表。

#####   Worker

每个Worker处理单个合并任务。合并任务是一个MapReduce的作业，其名称形式如下：<hostname>-compactor-<db>.<table>.<partition>。每个Worker向集群提交作业(如果定义了hive.compactor.jobs.queue)到Yarn队列，并等待作业完成。hive.compactor.worker.threads确定每个Metastore中的Worker数量。Hive数据仓库中的Worker总数决定了合并的最大并发数量。

#####  Cleaner

就是在合并后，确定不再需要增量文件之后删除增量文件的进程。

#####  AcidHouseKeeperService

此进程查找在hive.txn.timeout时间内没有心跳的事务，并中止它们。系统假设此时启动事务的客户端停止心跳、崩溃，而它锁定的资源应该被释放。

##### SHOW COMPACTIONS

此命令显示有关当前正在运行的合并和合并的近期历史记录(可配置保留期限)信息。从hive-12353开始，可显示合并的历史记录信息。

关于此命令和输出的更多信息，可参阅 [LanguageManual DDL#Show Compactions](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ShowCompactions)。影响该命令的输出的参数见“事务新的配置参数/合并历史记录”配置属性。系统保留每种类型的最后N个条目：失败、成功、尝试(其中N对每种类型都是可配置的)。



### 7 事务/锁管理器

Hive添加了一个名为“事务管理器”的新逻辑概念，它包含了以前的“数据库/表/分区锁管理器”的概念(hive.lock.Manager，缺省值为org.apache.hadoop.hive.ql.lockmgr.zookeeper.ZooKeeperHiveLockManager)。事务管理器现在还负责管理事务锁。默认的DummyTxnManager模仿老Hive版本的行为：没有事务，并使用hive.lock.Manager属性为表、分区和数据库创建锁管理器。新添加的DbTxnManager使用DbLockManager管理Hive Metastore中的所有锁/事务(事务和锁在服务器故障时是持久的)。这意味着在启用事务时，不再存在以前锁定Zookeeper中的行为了。为了避免客户端死掉并使事务或锁悬而未决，定期从锁持有者和事务发起者向Metastore发送心跳。如果在配置的时间内未接收到心跳，则锁或事务将被中止。

从Hive 1.3.0开始，DbLockManger可以通过控制hive.lock.numretires和hive.lock.sleep.between.retries来设定尝试获取锁的时间。当DbLockManager无法获得锁(由于存在竞争锁)时，它将退出，并在某个时间段后再试。为了支持短时间运行的即席查询，而又不对Metastore压力太大，每次重试之后，DbLockManager将等待时间加倍。初始回退时间为100 ms，并以hive.lock.sleep.between.retries为上限。hive.lock.numretries是它将重试请求锁的总次数。因此，调用获取锁将阻塞的总时间(给定100次重试和60次睡眠时间)是(100 ms 200 ms 400 ms.51200 ms 60s ..60s)=91分钟：42秒：300毫秒。

锁管理器中使用的锁的详细信息见[Show locks](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL#LanguageManualDDL-ShowLocks)。

请注意，使用DbTxnManager的锁管理器将获取所有表上的锁，即使是那些没有设置“transactional=true”属性的表。默认情况下，对非事务性表的INSERT操作将获得独占锁，从而阻止其他插入和读取。虽然在技术上是正确的，但这与传统Hive的工作方式(例如，w/o 锁管理器)是不同的, 为了向后兼容，提供了hive.txn.strict.locking.mode(见下表)模式，这将使该锁管理器在非事务性表的插入操作上获取共享锁。这恢复了以前的语义，同时仍然提供了锁管理器的好处，例如在读取表时防止表被删除。

请注意，对于事务表，插入总是获取共享锁，因为这些表在存储层实现了MVCC架构，并且即使存在并发修改操作，也能提供读的强一致性（快照隔离）。



### 8 事务参数配置汇总

配置一下属性用于开启事务支持，如果你只是学习可以配置，但是公司建议还是别开启了，在需要的时候只在会话里开启即可

```xml
<property>
    <name>hive.support.concurrency</name>
    <value>true</value>
</property>
<property>
    <name>hive.exec.dynamic.partition.mode</name>
    <value>nonstrict</value>
</property>
<property>
    <name>hive.txn.manager</name>
    <value>org.apache.hadoop.hive.ql.lockmgr.DbTxnManager</value>
</property>
<property>
    <name>hive.compactor.initiator.on</name>
    <value>true</value>
</property>
<property>
    <name>hive.compactor.worker.threads</name>
    <value>1</value>
</property>
<property>
    <name>hive.enforce.bucketing</name>
    <value>true</value>
</property>
```



许多新的配置参数已经被添加到系统，用以支持事务。

| **配置关键**                                                 | **默认值**     | **位置**                       | **注解**                                                     |
| ------------------------------------------------------------ | -------------- | ------------------------------ | ------------------------------------------------------------ |
| [hive.txn.manager](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.txn.manager) | 见注解         | 客户端/ HiveServer2            | 默认值： org.apache.hadoop.hive.ql.lockmgr.DummyTxnManager支持事务所需的值： org.apache.hadoop.hive.ql.lockmgr.DbTxnManager而DummyTxnManager实现Hive 0.13前的行为，并且不提供事务支持。 |
| [hive.txn.strict.locking.mode](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.txn.strict.locking.mode) | true           | 客户端/ HiveServer2            | 在严格模式非ACID资源使用标准的R / W锁语义，例如INSERT将获得排他锁；在非严格模式，对于非ACID资源，INSERT将只获取共享锁，它允许两个并发写入到相同的分区，但仍让锁管理器在表被写入时阻止DROP TABLE等（从Hive 2.2.0）。 |
| [hive.txn.timeout](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.txn.timeout) | 300            | 客户端/ HiveServer2 /Metastore | 如果客户没有发送心跳，多长时间（单位秒）之后宣布事务中止。对于所有组件/服务，至关重要的是此属性要有相同的值。（注5） |
| [hive.txn.heartbeat.threadpool.size](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.txn.heartbeat.threadpool.size) | 5              | 客户端/ HiveServer2            | 用于心跳的线程数（从[Hive 1.3.0和2.0.0](https://issues.apache.org/jira/browse/HIVE-12366)开始）。 |
| [hive.timedout.txn.reaper.start](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.timedout.txn.reaper.start) | 100秒          | Metastore                      | Metastore启动后，启动第一个收割器(中止超时事务的进程)的延迟时间（从Hive 0.13开始）。用于控制上述的AcidHouseKeeperServcie。 |
| [hive.timedout.txn.reaper.interval](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.timedout.txn.reaper.interval) | 180秒          | Metastore                      | 描述收割程序(中止超时事务的进程)运行频率的时间间隔。（从Hive 0.13开始）。用于控制上述的AcidHouseKeeperServcie。 |
| [hive.txn.max.open.batch](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.txn.max.open.batch) | 1000           | 客户                           | 可以在调用一次open_txns（）获取到事务的最大数量。（注1）     |
| [hive.max.open.txns](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.max.open.txns) | 100000         | HiveServer2 /Metastore         | 打开的事务的最大数量。如果当前打开的事务达到此限制，则将来打开的事务请求将被拒绝，直到该数目低于限制为止。(从Hive 1.3.0和2.1.0) |
| [hive.count.open.txns.interval](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.count.open.txns.interval) | 1秒            | HiveServer2 /Metastore         | 检查打开事务计数的时间间隔（单位秒）(从Hive 1.3.0和2.1.0开始)。 |
| [hive.txn.retryable.sqlex.regex](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.txn.retryable.sqlex.regex) | “”（空字符串） | HiveServer2 /Metastore         | 由逗号分隔的一组用于描述SQL状态，错误代码，可重试的SQLExceptions错误信息的正则表达式，这是适合于Hive Metastore数据库（从[Hive 1.3.0和2.1.0](https://issues.apache.org/jira/browse/HIVE-12637)）。例子见[配置属性](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.txn.retryable.sqlex.regex)。 |
| [hive.compactor.initiator.on](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.compactor.initiator.on) | false          | Metastore                      | 缺省值为false，支持事务的话需要为true。是否在此Metastore实例上运行启动器（initiator）和清理（cleaner）的线程。在Hive1.3.0之前，关键是要正好在一个独立的Metastore服务实例上启用它(尚未强制执行)。从Hive1.3.0开始，可以在任意数量的独立Metastore实例上启用此属性。 |
| [hive.compactor.worker.threads](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.compactor.worker.threads) | 0              | Metastore                      | 在这个metastore实例上运行多少个合并工作线程。（注2）默认值为0，支持事务时至少在一个Metastore实例上大于0。 |
| [hive.compactor.worker.timeout](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.compactor.worker.timeout) | 86400          | Metastore                      | 合并作业运行多长时间（秒）后会被宣告失败，并重新排队。       |
| [hive.compactor.cleaner.run.interval](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.compactor.cleaner.run.interval) | 5000           | Metastore                      | 运行清理线程的间隔（毫秒）。（[Hive 0.14.0](https://issues.apache.org/jira/browse/HIVE-8258)和更高版本）。 |
| [hive.compactor.check.interval](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.compactor.check.interval) | 300            | Metastore                      | 检查表或分区是否需要合并的时间间隔（单位秒）。（注3）        |
| [hive.compactor.delta.num.threshold](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.compactor.delta.num.threshold) | 10             | Metastore                      | 在表或分区引发次要/轻度/minor合并的差量目录数目。            |
| [hive.compactor.delta.pct.threshold](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.compactor.delta.pct.threshold) | 0.1            | Metastore                      | 触发一个主要/深度/major合并任务的增量文件相对基础文件大小的百分比。1 = 100％，默认0.1 = 10％。 |
| [hive.compactor.abortedtxn.threshold](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.compactor.abortedtxn.threshold) | 1000           | Metastore                      | 触发一个主要/深度/major合并任务的涉及给定表或分区的中止事务数。 |
| [hive.compactor.max.num.delta](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.compactor.max.num.delta) | 500            | Metastore                      | 在单个合并作业中试图处理增量文件的最大数量(从Hive1.3.0开始)。（注4） |
| [hive.compactor.job.queue](https://cwiki.apache.org/confluence/display/Hive/Configuration+Properties#ConfigurationProperties-hive.compactor.job.queue) | “”（空字符串） | Metastore                      | 用于指定将提交合并作业到Hadoop队列的名称。设置为空字符串，则由Hadoop选择队列(从Hive1.3.0开始)。 |
| 合并的历史记录                                               |                |                                |                                                              |
| hive.compactor.history.retention.succeeded                   | *3*            | Metastore                      | 要在历史记录中保留的成功合并条目的数量(每个分区)。           |
| hive.compactor.history.retention.failed                      | *3*            | Metastore                      | 要在历史记录中保留的合并失败条目的数量(每个分区)。           |
| hive.compactor.history.retention.attempted                   | *2*            | Metastore                      | 要在历史记录中保留的尝试合并条目的数量(每个分区)。           |
| hive.compactor.initiator.failed.compacts.threshold           | *2*            | Metastore                      | 对给定分区合并连续失败的数目，在此之后，启动器（Initiator）将停止自动调度合并。此时，仍然可以使用ALTER TABLE来启动合并。一旦手动启动的合并成功，就恢复自动启动的合并。请注意，这个值必须小于hive.compactor.history.retention.failed。 |
| hive.compactor.history.reaper.interval                       | *2M*           | Metastore                      | 控制清除compactions历史记录的进程多久运行一次。              |

 注1： hive.txn.max.open.batch控制流代理(如Flume或Storm)同时打开的事务。然后，流代理将该数量的条目写入单个文件(每个Flume代理或Storm bolt)。因此，增加此值会减少流代理创建的增量文件的数量。但它也会增加Hive在任何给定时间必须跟踪的已打开事务的数量，这可能会对读取性能产生负面影响。

注2：工作线程生成MapReduce作业以执行合并。它们自己不执行合并。增加工作者线程的数量将减少需要合并的表或分区的时间。它也将增加Hadoop集群上的后台负载，因为更多MapReduce作业要在后台运行。每次合并都可以处理一个分区(如果没有分区，则是整个表)。

注3： 减小该值将减少需要合并的表或分区启动合并所需的时间。然而，检查是否需要合并需要对每个表或分区调用NameNode，以确认每个表或分区自上一次主要/深度/major合并以来是否进行了事务处理。因此，降低此值将增加NameNode的负载。

注4： 如果合并程序检测到有非常多的增量文件，它就首先运行几个小的部分合并(在当前顺序中)，然后执行实际请求的合并。

注5： 如果该值不是相同的，则活动事务可能被确定为“timed out（超时）”并因此中止。这还将导致诸如“没有这样的事务（No such transaction）...”、“没有这样的锁（No such lock）...”之类的错误。



```sql
CREATE TABLE table_name (
  id                int,
  name              string
)
CLUSTERED BY (id) INTO 2 BUCKETS STORED AS ORC
TBLPROPERTIES ("transactional"="true",
  "compactor.mapreduce.map.memory.mb"="2048",                   -- 指定合并map作业的属性
  "compactorthreshold.hive.compactor.delta.num.threshold"="4",  -- 如果有超过4个增量目录，则触发轻度合并
  "compactorthreshold.hive.compactor.delta.pct.threshold"="0.5" -- 如果增量文件的大小与基础文件的大小的比率大于50%，则触发深度合并
);
```

 例如：在TBLPROPERTIES中设置请求级别的合并选项

```sql
ALTER TABLE table_name COMPACT 'minor'
   WITH OVERWRITE TBLPROPERTIES ("compactor.mapreduce.map.memory.mb"="3072");  -- 指定合并map作业的属性

ALTER TABLE table_name COMPACT 'major'
   WITH OVERWRITE TBLPROPERTIES ("tblprops.orc.compress.size"="8192");         -- 更改任何其他Hive表属性
```





## 总结

1. 所有 INSERT 语句都会创建 `delta` 目录。UPDATE 语句也会创建 `delta` 目录，但会先创建一个 `delete` 目录，即先删除、后插入。`delete` 目录的前缀是 delete_delta； 
2. 每个事务的文件夹，都有两个文件，一个是数据文件，一个事务的信息文件