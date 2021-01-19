# Hadoop 高可用

在Hadoop 2.0以前的版本，NameNode面临单点故障风险（SPOF），也就是说，一旦NameNode节点挂了，整个集群就不可用了，而且需要借助辅助NameNode来手工干预重启集群，这将延长集群的停机时间。而Hadoop 2.0版本支持一个备用节点用于自动恢复NameNode故障，Hadoop 3.0则支持多个备用NameNode节点，这使得整个集群变得更加可靠。

## 什么是 Hadoop 高可用

Hadoop 2.0版本支持一个备用节点用于自动恢复NameNode故障，Hadoop 3.0则支持多个备用NameNode节点，这些都是针对NameNode单点故障问题比较可靠的解决方案。利用一个或者多个备用NameNode来达到故障时自动切换的目的，这就是我们所说的Hadoop高可用。

### 什么是故障切换

当系统中其中一项设备失效而无法运作时，另一项设备即可自动接手原失效系统所执行的工作，这就是故障切换。

故障切换可分成两种方式：

**优雅的故障切换**：这种需要系统管理员手动切换。通常在定期维护系统的时候使用这种故障切换方式，也就是说必须人工干预，把集群控制权切换到备用的NameNode。
**自动故障切换**：系统自动把集群控制器切换到备用NameNode，且切换过程不需要人工干预。

## NameNode 高可用

Hadoop 实现自动故障切换需要用到下面的组件：

- ZooKeeper quorum
- ZKFailoverController 进程（ZKFC）

### ZooKeeper quorum

ZooKeeper quorum 是一种集中式服务，主要为分布式应用提供协调、配置、命名空间等功能。它提供组服务和数据同步服务，它让客户端可以实时感知数据的更改，并跟踪客户端故障。HDFS故障自动切换的实现依赖下面两个方面：

- **故障监测**：ZooKeeper维护一个和NameNode之间的会话。如果NameNode发生故障，该会话就会过期，会话一旦失效了，ZooKeeper将通知其他NameNode启动故障切换进程。
- **活动NameNode选举**：ZooKeeper提供了一种活动节点选举机制。只要活动的NameNode发生故障失效了，其他NameNode将从ZooKeeper获取一个排它锁，并把自身声明为活动的NameNode。

### ZKFailoverController（ZKFC）

ZKFC 是 ZooKeeper 的监控和管理 namenode 的一个客户端。所以每个运行 namenode 的机器上都会有 ZKFC。

那ZKFC具体作用是什么？主要有以下3点：

**状态监控**：ZKFC 会定期用 ping 命令监测活动的 NameNode，如果 NameNode 不能及时响应ping 命令，那么 ZooKeeper 就会判断该活动的 NameNode 已经发生故障了。

**ZooKeeper会话管理**：如果 NameNode 是正常的，那么它和 ZooKeeper 会保持一个会话，并持有一个 znode 锁。如果会话失效了，那么该锁将自动释放。

**基于ZooKeeper的选举**：如果 NameNode 是正常的，ZKFC 知道当前没有其他节点持有 znode 锁，那么 ZKFC 自己会试图获取该锁，如果锁获取成功，那么它将赢得选举，并负责故障切换工作。这里的故障切换过程其实和手动故障切换过程是类似的；先把之前活动的节点进行隔离，然后把 ZKFC 所在的机器变成活动的节点。