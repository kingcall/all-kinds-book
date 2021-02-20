# HDFS 磁盘均衡

## HDFS 磁盘均衡器

HDFS 提供了一个用于 Datanode 内多磁盘之间的数据均衡工具，即 Diskbalancer （磁盘均衡器），它把数据均衡的分发到一个 Datanode 下的多个磁盘。Diskbalancer 和 Hadoop 2.0 版本以前提供的 Balancer 不同，因为 Balancer 关心的是不同 Datanode 之间的数据均衡，Datanode 内多个磁盘的数据均衡它是不起作用的。

HDFS 由于以下原因，在把数据存储到 Datanode 多个磁盘的时候，会出现磁盘之间数据不均衡的情况：

- 大量的数据写入和删除
- 磁盘更换

上面这两点可能导致数据在 Datanode 内的多个磁盘发生明显倾斜。这种情况现有的 HDFS balancer 均衡工具没办法处理，上面说了，它只关心 Datanode 之间的数据均衡，所以，Hadoop 3.0 提供了 Diskbalancer 工具，用于均衡一个Datanode 内多个磁盘之间的数据均衡。

## Diskbalancer

Hadoop HDFS balancer 工具通过创建一个计划（命令集）并在 Datanode 执行该计划来工作。这里的计划主要描述的是有多少数据需要在磁盘之间做迁移。一个计划有很多迁移步骤，比如，源磁盘，目标磁盘和需要迁移的字节数。计划可以针对某一个 Datanode 执行特定操作。默认情况下，Diskbalancer 是未启用状态，您可以在 hdfs-site.xml 配置文件把 dfs.disk.balancer.enabled 设置为 true 来启用它。

当我们往 HDFS 上写入新的数据块，DataNode 将会使用 volume 选择策略来为这个块选择存储的地方。目前 Hadoop 支持两种 volume 选择策略：round-robin（循环策略） 和 available space（可用空间策略），选择哪种策略我们可以通过下面的参数来设置。

```
dfs.datanode.fsdataset.volume.choosing.policy
```

- 循环策略：将新块均匀分布在可用磁盘上。
- 可用空间策略：它是优先将数据写入具有最大可用空间的磁盘（通过百分比计算的）。

![HDFS磁盘均衡](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570195566000_20191004212607902952.png)

默认情况下，DataNode 是使用基于 round-robin 策略来写入新的数据块。然而在一个长时间运行的集群中，由于 HDFS 中的大规模文件删除或者通过往 DataNode 中添加新的磁盘，仍然会导致同一个 DataNode 中的不同磁盘存储的数据很不均衡。即使你使用的是基于可用空间的策略，卷（volume）不平衡仍可导致较低效率的磁盘I/O。比如所有新增的数据块都会往新增的磁盘上写，在此期间，其他的磁盘会处于空闲状态，这样新的磁盘将会是整个系统的瓶颈。

Apache Hadoop 社区之前开发了几个离线脚本来解决磁盘不均衡的问题。然而，这些脚本都是在HDFS代码库之外，在执行这些脚本往不同磁盘之间移动数据的时候，需要要求 DataNode 处于关闭状态。结果，HDFS-1312 还引入了一个在线磁盘均衡器，旨在根据各种指标重新平衡正在运行 DataNode 上的磁盘数据。和现有的 HDFS 均衡器类似，HDFS 磁盘均衡器在 DataNode 中以线程的形式运行，并在相同存储类型的卷（volumes）之间移动数据。注意，本文介绍的HDFS 磁盘均衡器是在同一个 DataNode 中的不同磁盘之间移动数据，而之前的 HDFS 均衡器是在不同的 DataNode 之间移动数据。

## Diskbalancer 的使用

我们通过一个例子逐步探讨如何使用 Diskbalancer。

首先，确保所有 DataNode 上的 `dfs.disk.balancer.enabled` 参数设置成 `true`。本例子中，我们的 DataNode 已经挂载了一个磁盘（/mnt/disk1），现在我们往这个DataNode 上挂载新的磁盘（/mnt/disk2），我们使用 df命令来显示磁盘的使用率：

```
# df -h
….
/var/disk1      5.8G  3.6G  1.9G  66% /mnt/disk1
/var/disk2      5.8G   13M  5.5G   1% /mnt/disk2
```

从上面的输出可以看出，两个磁盘的使用率很不均衡，所以我们来将这两个磁盘的数据均衡一下。典型的磁盘平衡器任务涉及三个步骤（通过 HDFS 的 diskbalancer 命令）：`plan`, `execute` 和 `query`。

第一步，HDFS 客户端从 NameNode 上读取指定 DataNode 的必要信息以生成执行计划：

```
# hdfs diskbalancer -plan lei-dn-3.example.org
16/08/19 18:04:01 INFO planner.GreedyPlanner: Starting plan for Node : lei-dn-3.example.org:20001
16/08/19 18:04:01 INFO planner.GreedyPlanner: Disk Volume set 03922eb1-63af-4a16-bafe-fde772aee2fa Type : DISK plan completed.
16/08/19 18:04:01 INFO planner.GreedyPlanner: Compute Plan for Node : lei-dn-3.example.org:20001 took 5 ms
16/08/19 18:04:01 INFO command.Command: Writing plan to : /system/diskbalancer/2016-Aug-19-18-04-01
```

从上面的输出可以看出，HDFS 磁盘平衡器通过使用 DataNode 报告给 NameNode 的磁盘使用信息，并结合计划程序来计算指定 DataNode 上数据移动计划的步骤，每个步骤指定要移动数据的源卷和目标卷，以及预计移动的数据量。

截止到撰写本文的时候，HDFS仅仅支持 GreedyPlanner，其不断地将数据从最常用的设备移动到最少使用的设备，直到所有数据均匀地分布在所有设备上。用户还可以在使用 plan 命令的时候指定空间利用阀值，也就是说，如果空间利用率的差异低于此阀值，planner 则认为此磁盘已经达到了平衡。当然，我们还可以通过使用 —bandwidth 参数来限制磁盘数据移动时的I/O。

磁盘平衡执行计划生成的文件内容格式是 Json 的，并且存储在 HDFS 之上。在默认情况下，这些文件是存储在 /system/diskbalancer 目录下面：

```
$ hdfs dfs -ls /system/diskbalancer/2016-Aug-19-18-04-01
Found 2 items
-rw-r--r--   3 hdfs supergroup       1955 2016-08-19 18:04 /system/diskbalancer/2016-Aug-19-18-04-01/lei-dn-3.example.org.before.json
-rw-r--r--   3 hdfs supergroup        908 2016-08-19 18:04 /system/diskbalancer/2016-Aug-19-18-04-01/lei-dn-3.example.org.plan.json
```

可以通过下面的命令在 DataNode 上执行这个生成的计划：

```
$ hdfs diskbalancer -execute /system/diskbalancer/2016-Aug-17-17-03-56/172.26.10.16.plan.json
16/08/17 17:22:08 INFO command.Command: Executing "execute plan" command
```

这个命令将 JSON 里面的计划提交给 DataNode，而 DataNode 会启动一个名为 BlockMover 的线程中执行这个计划。我们可以使用 query 命令来查询 DataNode 上diskbalancer 任务的状态：

```
# hdfs diskbalancer -query lei-dn-3:20001
16/08/19 21:08:04 INFO command.Command: Executing "query plan" command.
Plan File: /system/diskbalancer/2016-Aug-19-18-04-01/lei-dn-3.example.org.plan.json
Plan ID: ff735b410579b2bbe15352a14bf001396f22344f7ed5fe24481ac133ce6de65fe5d721e223b08a861245be033a82469d2ce943aac84d9a111b542e6c63b40e75
Result: PLAN_DONE
```

上面结果输出的 PLAN_DONE 表示 disk-balancing task 已经执行完成。为了验证磁盘平衡器的有效性，我们可以使用 df -h 命令来查看各个磁盘的空间使用率：

```
# df -h
Filesystem      Size  Used Avail Use% Mounted on
….
/var/disk1      5.8G  2.1G  3.5G  37% /mnt/disk1
/var/disk2      5.8G  1.6G  4.0G  29% /mnt/disk2
```

上面的结果证明，磁盘平衡器成功地将 /var/disk1 和 /var/disk2 空间使用率的差异降低到 10% 以下，说明任务完成！

