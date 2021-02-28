[TOC]



## 什么是Hadoop

Hadoop 是使用 Java 编写，允许分布在集群，使用简单的编程模型的处理大型数据集处理的Apache 的开源框架。 Hadoop 框架提供跨计算机集群的分布式存储和计算的环境。 Hadoop 是专为从单一服务器到上千台机器扩展，每个机器都可以提供本地计算和存储。

Hadoop 可以用单节点模式安装，但是只有多节点集群才能发挥 Hadoop 的优势，我们可以把集群扩展到上千个节点，而且扩展过程中不需要先停掉集群。

**Hadoop 由三个关键部分组成：**
**HDFS**：Hadoop 分布式文件系统，它是 Hadoop 数据存储层。
**MapReduce**：数据计算框架
**YARN**：负责资源管理和任务调度。

## Hadoop 架构

在了解了什么是 Hadoop 之后，再来详细了解一下 Hadoop 的架构。
![Hadoop架构图](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/Hadoop-Architecture_20191001204653970236.png)

Hadoop 以主从的方式工作。一个 Master 节点和多个 Slave 节点，slave 节点可以扩招到1000个。Master 节点管理，维护和监控 slave 节点， slave 节点是真正在干活的机器。Master 节点应该部署在配置较高的机器上面，因为它是 hadoop 集群的核心。Maste r存储元数据（即数据的数据），slave 是真正存储数据的机器。客户端通过master 节点来分发任务。

## Hadoop组件

在本教程，我们将会学习到 HDFS，MapReduce 和Yarn 这3大重要组件。

### HDFS是什么

**HDFS** 即 Hadoop 分布式文件系统（Hadoop Distribute File System），以分布式存储的方式存储数据。

在 Hadoop 架构里面，master 节点上会运行一个叫做 namenode 的守护进程，每个 slave 节点上都会有 datanode 守护进程，两个进程都是属于HDFS 的。因此，slave 节点也叫做 datanode 节点。Namenode 主要用于存储元数据和管理 datanode 节点。而 datanode 则是真正存储数据和执行任务的地方。
![HDFS架构图](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/HDFS-Architecture_20191001204831887472.png)

HDFS 是一个具有故障容错，分布式的，高可靠的且可扩展的用于数据存储的文件系统。HDFS 是为了存储海量数据而开发的，数据量可达到 TB 级别。数据文件会被分割成块（默认一个块大小为128MB）并存储在多个节点。分割的数据块按照复制因子进行跨 datanode 复制。避免 datanode 节点发生故障后造成数据丢失。假如有一个文件大小为640MB，那么它将会被分割成5个块，每个块128MB（按照默认的数据块大小）。

### MapReduce是什么

**Hadoop MapReduce** 是一种编程模型，它是 Hadoop 最重要的组件之一。它用于计算海量数据，并把计算任务分割成许多在集群并行计算的独立运行的 task。MapReduce 是 Hadoop 的核心，它会把计算任务移动到离数据最近的地方进行执行，因为移动大量数据是非常耗费资源的。因此，MapReduce 是一个分布式处理海量数据的计算框架。由于数据存储以分布式方式存储在 HDFS，它为 MapReduce 执行并行任务提供了基础。

### Yarn是什么

Yarn 是一种资源管理系统，在集群模式下，管理、分配和释放资源（CPU，内存，磁盘）变得非常复杂。而 Yarn 可以非常高效的管理这些资源。它根据来自任何应用程序的请求分配相同的值。在 Master 节点会运行一个叫ResourceManager 守护进程，且每个slave 节点都会有一个叫 NodeManager 的守护进程。

## Hadoop 守护进程

守护进程是一种运行在后台的进程。Hadoop 主要有4个守护进程。
![Yarn守护进程](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/HADOOP-DAEMONS-ARCHITECTURE_20191001213041847295.png)
NameNode ：它是HDFS运行在Master节点守护进程。

- **DataNode**：它是 HDFS 运行在Slave节点守护进程。
- **ResourceManager**：它是 Yarn 运行在 Master 节点守护进程。
- **NodeManager**：它是 Yarn 运行在 Slave 节点的守护进程。

除了这些，可能还会有 secondary NameNode，standby NameNode，Job HistoryServer 等进程。



### secondary NameNode

#### NameNode 机制介绍

很多人都认为，Secondary NameNode是NameNode的备份，是为了防止NameNode的单点失败的,其实不是的。

在Hadoop中，有一些命名不好的模块，Secondary NameNode是其中之一。从它的名字上看，它给人的感觉就像是NameNode的备份。但它实际上却不是。很多Hadoop的初学者都很疑惑，Secondary NameNode究竟是做什么的，而且它为什么会出现在HDFS中

从它的名字来看，你可能认为它跟NameNode有点关系。没错，你猜对了。因此在我们深入了解Secondary NameNode之前，前面我们已经了解了NameNode是做什么的。

NameNode主要是用来保存HDFS的元数据信息，比如命名空间信息，块信息等。当它运行的时候，这些信息是存在内存中的。但是这些信息也可以持久化到磁盘上。

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/53536bcc0cf2bb589c5e16d0.png)

上面的这张图片展示了NameNode怎么把元数据保存到磁盘上的。这里有两个不同的文件：

1. fsimage - 它是在NameNode启动时对整个文件系统的快照
2. edit logs - 它是在NameNode启动后，对文件系统的改动序列

只有在NameNode重启时，edit logs才会合并到fsimage文件中，从而得到一个文件系统的最新快照。但是在产品集群中NameNode是很少重启的，这也意味着当NameNode运行了很长时间后，edit logs文件会变得很大。在这种情况下就会出现下面一些问题：

1. edit logs文件会变的很大，怎么去管理这个文件是一个挑战。
2. NameNode的重启会花费很长时间，因为有很多改动在edit logs中要合并到fsimage文件上。
3. 如果NameNode挂掉了，那我们就丢失了很多改动因为此时的fsimage文件非常旧。[笔者认为在这个情况下丢失的改动不会很多, 因为丢失的改动应该是还在内存中但是没有写到edit logs的这部分]

因此为了克服这个问题，我们需要一个易于管理的机制来帮助我们减小edit logs文件的大小和得到一个最新的fsimage文件，这样也会减小在NameNode上的压力。这跟Windows的恢复点是非常像的，Windows的恢复点机制允许我们对OS进行快照，这样当系统发生问题时，我们能够回滚到最新的一次恢复点上。

现在我们明白了NameNode的功能和所面临的挑战 - 保持文件系统最新的元数据。那么，这些跟Secondary NameNode又有什么关系呢？



> NameNode是什么时候将改动写到edit logs中的？这个操作实际上是由DataNode的写操作触发的，当我们往DataNode写文件时，DataNode会跟NameNode通信，告诉NameNode什么文件的第几个block放在它那里，NameNode这个时候会将这些元数据信息写到edit logs文件中。

#### Secondary NameNode 的解决方案

SecondaryNameNode就是来帮助解决上述问题的，它的职责是合并NameNode的edit logs到fsimage文件中。

![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/535371590cf2bb589c5e2391.png)

上面的图片展示了Secondary NameNode是怎么工作的。

1. 首先，它定时到NameNode去获取edit logs，并更新到fsimage上。[Secondary NameNode自己的fsimage]
2. 一旦它有了新的fsimage文件，它将其拷贝回NameNode中。
3. NameNode在下次重启时会使用这个新的fsimage文件，从而减少重启的时间。

Secondary NameNode的整个目的是在HDFS中提供一个检查点。它只是NameNode的一个助手节点。这也是它在社区内被认为是检查点节点的原因。

现在，我们明白了Secondary NameNode所做的不过是在文件系统中设置一个检查点来帮助NameNode更好的工作。它不是要取代掉NameNode也不是NameNode的备份。所以从现在起，让我们养成一个习惯，称呼它为检查点节点吧。



####  Secondary Namenode工作原理

1. SecondaryNameNode通知NameNode准备提交edits文件，此时主节点产生edits.new
2. SecondaryNameNode通过http get方式获取NameNode的fsimage与edits文件（在SecondaryNameNode的current同级目录下可见到 temp.check-point或者previous-checkpoint目录，这些目录中存储着从namenode拷贝来的镜像文件）
3. SecondaryNameNode开始合并获取的上述两个文件，产生一个新的fsimage文件fsimage.ckpt
4. SecondaryNameNode用http post方式发送fsimage.ckpt至NameNode
5. NameNode将fsimage.ckpt与edits.new文件分别重命名为fsimage与edits，然后更新fstime，整个checkpoint过程到此结束。 在新版本的hadoop中（hadoop0.21.0）,SecondaryNameNode两个作用被两个节点替换， checkpoint node与backup node. SecondaryNameNode备份由三个参数控制fs.checkpoint.period控制周期，fs.checkpoint.size控制日志文件超过多少大小时合并， dfs.http.address表示http地址，这个参数在SecondaryNameNode为单独节点时需要设置。

#### 相关配置文件

core-site.xml：这里有2个参数可配置，但一般来说我们不做修改。fs.checkpoint.period表示多长时间记录一次hdfs的镜像。默认是1小时。fs.checkpoint.size表示一次记录多大的size，默认64M。

```
<property><name>fs.checkpoint.period</name>
<value>3600</value>

<description>The number of seconds between two periodic checkpoints.

</description>

</property>

 

<property>

<name>fs.checkpoint.size</name>

<value>67108864</value>

<description>The size of the current edit log (in bytes) that triggers

a periodic checkpoint even if the fs.checkpoint.period hasn’t expired.

</description>

</property>
```



### standby NameNode

### Job HistoryServer



## Hadoop是怎么工作的

Apache Hadoop 工作原理：

1. 输入数据被划分成若干个128MB（默认值）的块，然后把它们移动到不同的节点。
2. 在多个 datanode 存储完所有数据块之后，用户才能处理这些数据。
3. 接着，master 把用户提交的程序调度到独立的节点上。
4. 等所有节点处理完数据之后，输出计算结果并写回 HDFS。

## Hadoop生态下的组件介绍

在本节我们会涉及到 Hadoop 生态下的各种组件。先看看 Hadoop 生态下有哪些组件：
![Hadoop生态](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/hadoop-ecosystem_20191001213451911294.png)
Hadoop HDFS：Hadoop 分布式存储系统。
Yarn：Hadoop 2.x版本开始才有的资源管理系统。
MapReduce：并行处理框架。
HBase：基于 HDFS 的列式存储数据库，它是一种 NoSQL 数据库，非常适用于存储海量的稀疏的数据集。
Hive：Apache Hive 是一个数据仓库基础工具，它适用于处理结构化数据。它提供了简单的 sql 查询功能，可以将 sql 语句转换为 MapReduce 任务进行运行。
Pig：它是一种高级脚本语言。利用它不需要开发 Java 代码就可以写出复杂的数据处理程序。
Flume：它可以从不同数据源高效实时的收集海量日志数据。
Sqoop：适用于在 Hadoop 和关系数据库之间抽取数据。
Oozie：这是一种 Java Web 系统，用于 Hadoop 任务的调度，例如设置任务的执行时间和执行频率等。
Zookeeper：用于管理配置信息，命名空间。提供分布式同步和组服务。
Mahout：可扩展的机器学习算法库。