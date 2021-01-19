# HDFS Federation（联邦机制）

HDFS Federation 中文意思为 HDFS 联盟或者 HDFS 联邦。这里并非指多个集群，更准确的应该是一个集群有多个命名空间，即多个 Namenode。

## Hadoop 1.0 HDFS 架构

Hadoop HDFS 有两个主要的分层：HDFS包含两个层次：**命名空间管理**（Namespace） 和 **块/存储管理**（Block Storage）。

![Hadoop1.0 HDFS 架构](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570194530000_20191004210851503478.png)

- 命名空间管理（Namespace）
  HDFS的命名空间包含目录、文件和块。命名空间管理是指命名空间支持对HDFS中的目录、文件和块做类 似文件系统的创建、修改、删除、列表文件和目录等基本操作。
- 块/存储管理（Block Storage）
  在块存储服务中包含两部分工作：块管理和物理存储。这是一个更通用的存储服务。其他的应用可以直接建立在Block Storage上，如HBase，Foreign Namespaces等。
  - 块管理
    - 处理 DataNode 向 NameNode 注册的请求，处理 Datanode 的成员关系，接受来自 DataNode 周期性的心跳。
    - 处理来自块的报告信息，维护块的位置信息。
    - 处理与块相关的操作：块的创建、删除、修改及获取块信息。
    - 管理副本放置（replica placement）和块的复制及多余块的删除。
  - 物理存储
    所谓物理存储就是：DataNode 把块存储到本地文件系统中，对本地文件系统的读、写。

## Hadoop 1.0 HDFS 架构的局限性

Hadoop 1.0 HDFS 架构只允许整个集群中存在一个 namespace，而该 namespace 被仅有的一个 namenode 管理。这个架构使得 HDFS 非常容易实现，但是，它在具体实现过程中耦合度比较高，进而导致了很多局限性。

HDFS的局限性主要为：

- 块存储和 namespace 高耦合
  当前 namenode 中的 namespace 和 block management 的结合使得这两层架构耦合在一起，难以让其他可能namenode 实现方案直接使用 block storage。
- namenode 扩展性
  HDFS 的底层存储，即 Datanode 节点是可以水平扩展的，但 namespace 不可以。当前的 namespace 只能存放在单个namenode 上，而 namenode 在内存中存储了整个分布式文件系统中的元数据信息，这限制了集群中数据块，文件和目录的数目。
- 性能
  文件操作的性能制约于单个 namenode 的吞吐量，单个 namenode 当前仅支持约 60,000 个并发 task，而下一代Apache MapReduce 将支持超过 1,00,000 个并发任务，这意味着将需要更多的 Namenode 。
- 隔离性
  现在大部分公司的集群都是共享的，每天有来自不同部门的不同用户提交作业。单个 namenode 难以提供隔离性，即：某个用户提交的负载很大的 job 会减慢其他用户的 job ，单一的 namenode 难以像 HBase 按照应用类别将不同作业分派到不同 namenode 上。

需要注意的，HDFS Federation 并不能解决单点故障问题，也就是说，每个 NameNode 都存在在单点故障问题，你需要为每个 namenode 部署一个备用 namenode 以应对 NameNode 挂掉对业务产生的影响。

## HDFS Federation 架构

为了解决上述第二节提到的问题，Hadoop 2.0引入了基于共享存储的高可用解决方案和 HDFS Federation。

![Hdfs Federation 联邦架构](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570194831000_20191004211352879340-20210111224242238.png)

为了水平扩展 namenode，federation 使用了多个独立的 namenode / namespace。这些 namenode 之间是联合的，也就是说，他们之间相互独立且不需要互相协调，各自分工，管理自己的区域。分布式的 datanode 被用作通用的数据块存储存储设备。每个 datanode 要向集群中所有的 namenode 注册，且周期性地向所有 namenode 发送心跳和块报告，并执行来自所有 namenode 的命令。

一个 block pool 由属于同一个 namespace 的数据块组成，每个 datanode 可能会存储集群中所有 block pool 的数据块。

每个 block pool 内部自治，也就是说各自管理各自的 block，不会与其他 block pool 交互。一个 namenode 挂掉了，不会影响其他 namenode。

某个 namenode 上的 namespace 和它对应的 block pool 一起被称为 namespace volume（命名空间卷）。它是管理的基本单位。当一个 namenode / nodespace 被删除后，其所有 datanode 上对应的 block pool 也会被删除。当集群升级时，每个 namespace volume 作为一个基本单元进行升级。

### 命名空间管理

Federation 中存在多个命名空间，如何划分和管理这些命名空间非常关键。在 Federation 中并没有采用“文件名 hash ”的方法，因为该方法的本地性非常差，比如：查看某个目录下面的文件，如果采用文件名 hash 的方法存放文件，则这些文件可能被放到不同 namespace 中，HDFS 需要访问所有 namespace，代价过大。为了方便管理多个命名空间，HDFS Federation 采用了经典的 Client Side Mount Table。

![hdfs federation（联邦）命名空间管理](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570194928000_20191004211529957523-20210111224247870.png)

如上图所示，下面四个深色三角形代表一个独立的命名空间，上方浅色的三角形代表从客户角度去访问的子命名空间。各个深色的命名空间 Mount 到浅色的表中，客户可以访问不同的挂载点来访问不同的命名空间，这就如同在 Linux 系统中访问不同挂载点一样。

这就是 HDFS Federation 中命名空间管理的基本原理：将各个命名空间挂载到全局 mount-table 中，就可以做将数据到全局共享；同样的命名空间挂载到个人的 mount-table 中，这就成为应用程序可见的命名空间视图。

### Block pool（块池）

所谓 Block pool （块池）就是属于单个命名空间的一组 block（块）。每一个 datanode 为所有的 block pool 存储块。Datanode 是一个物理概念，而 block pool 是一个重新将 block 划分的逻辑概念。同一个 datanode 中可以存着属于多个 block pool 的多个块。Block pool 允许一个命名空间在不通知其他命名空间的情况下为一个新的 block 创建 Block ID。同时，一个 Namenode 失效不会影响其下的 datanode 为其他 Namenode 的服务。

当 datanode 与 Namenode 建立联系并开始会话后自动建立 Block pool。每个 block 都有一个唯一的标识，这个标识我们称之为扩展的块 ID（ Extended Block ID ）= BlockID+BlockID。这个扩展的块 ID 在 HDFS 集群之间都是唯一的，这为以后集群归并创造了条件。

Datanode 中的数据结构都通过块池 ID（Block Pool ID）索引，即 datanode 中的 BlockMap，storage 等都通过BPID 索引。

在 HDFS 中，所有的更新、回滚都是以 Namenode 和 BlockPool 为单元发生的。即同一 HDFS Federation 中不同的 Namenode / BlockPool 之间没有什么关系。

## HDFS Federation 配置介绍

本节不会介绍具体的 namenode 和 datanode 的配置方法，而是重点介绍 HDFS 客户端配置方法，并通过对客户端配置的讲解让大家深入理解 HDFS Federation 引入的 “client-side mount table”（viewfs）这一概念，这是通过新的文件系统 viewfs 实现的。

### Hadoop 1.0中的配置

在 Hadoop 1.0 中，只存在一个 NameNode，所以，客户端设置 NameNode 的方式很简单，只需在 core-site.xml 中进行以下配置：

```
<property>
    <name>fs.default.name</name>
    <value>hdfs://host0001:9000</value>
</property>
```

设置该参数后，当用户使用以下命令访问 hdfs 时，目录或者文件路径前面会自动补上“hdfs://host0001:9000”：

```
bin/hadoop fs –ls /home/dongxicheng/data
```

其中 “/home/dongxicheng/data” 将被自动替换为 “hdfs://host0001:9000/home/dongxicheng/data” 。

当然，你也可以不在 core-site.xml 文件中配置 fs.default.name 参数，这样当你读写一个文件或目录时，需要使用全URI地址，即在前面添加 “hdfs://host0001:9000”，比如：

```
bin/hadoop fs –ls hdfs://host0001:9000/home/dongxicheng/data
```

### Hadoop 2.0 中的配置

在 Hadoop 2.0 中，由于引入了 HDFS Federation，当你启用该功能时，会同时存在多个可用的 namenode，为了便于配置 “fs.default.name”，你可以规划这些 namenode 的使用方式，比如图片组使用 namenode1，爬虫组使用 namenode2 等等，这样，爬虫组员工使用的 HDFS client 端的 core-site.xml 文件可进行如下配置：

```
<property>
    <name>fs.default.name</name>
    <value>hdfs://namenode1:9000</value>
 </property>
```

图片组员工使用的 HDFS client 端的 core-site.xml 文件可进行如下配置：

```
<property>
    <name>fs.default.name</name>
    <value>hdfs://namenode2:9000</value>
 </property>
```

从 HDFS 和 HBase 使用者角度看，当仅仅使用单 NameNode 上管理的数据时，是没有问题的。但是，当考虑 HDFS 之上的计算类应用，比如 YARN / MapReduce 应用程序，则可能出现问题。因为这类应用可能涉及到跨 NameNode 数据读写，这样必须显式的指定全URI，即输入输出目录中必须显式的提供类似 “hdfs://namenode2:9000” 的前缀，以注明目录管理者 NameNode 的访问地址。比如：

```
distcp hdfs://nnClusterY:port/pathSrc hdfs://nnCLusterZ:port/pathDest
```

为了解决这种麻烦，为用户提供统一的全局 HDFS 访问入口，HDFS Federation 借鉴 Linux 提供了 client-side mount table，这是通过一层新的文件系统 viewfs 实现的，它实际上提供了一种映射关系，将一个全局（逻辑）目录映射到某个具体的 namenode（物理）目录上，采用这种方式后，core-site.xml 配置如下：

```
<configuration xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include href="mountTable.xml"/>
<property>
<name>fs.default.name</name>
<value>viewfs://ClusterName/</value>
</property>
</configuration>
```

其中，“ClusterName” 是 HDFS 整个集群的名称，你可以自己定义一个。mountTable.xml 配置了全局（逻辑）目录与具体 namenode（物理）目录的映射关系，你可以类比 linux 挂载点来理解。

假设你的集群中有三个 namenode，分别是 namenode1，namenode2 和 namenode3，其中，namenode1 管理 /usr 和 /tmp 两个目录，namenode2 管理 /projects/foo 目录，namenode3 管理 /projects/bar 目录，则可以创建一个名为 “cmt” 的 client-side mount table，并在 mountTable.xml 中进行如下配置：

```
<configuration> 
<property> 
<name>fs.viewfs.mounttable.cmt.link./user</name> 
<value> hdfs://namenode1:9000/user </value> 
</property> 
<property>
<name>fs.viewfs.mounttable.cmt.link./tmp</name> 
<value> hdfs:/ namenode1:9000/tmp </value> 
</property> 
<property> 
<name>fs.viewfs.mounttable.cmt.link./projects/foo</name> 
<value> hdfs://namenode2:9000/projects/foo </value> 
</property> 
<property> 
<name>fs.viewfs.mounttable.cmt.link./projects/bar</name> 
<value> hdfs://namenode3:9000/projects/bar</value> 
</property> 
</configuration>
```

经过以上配置后，你可以像 1.0 那样，访问 HDFS 上的文件，比如：

```
bin/hadoop fs –ls /usr/dongxicheng/data
```

中的 “/usr/dongxicheng/data” 将被映射成 “hdfs://namenode1:9000/user/dongxicheng/data”。Client-side mount table 的引入为用户使用 HDFS 带来极大的方便，尤其是跨 namenode 的数据访问。