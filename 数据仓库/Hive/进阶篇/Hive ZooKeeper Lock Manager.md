Hive一直使用ZooKeeper作为分布式锁定管理器来支持HiveServer2中的并发。 基于ZooKeeper的锁管理器在小型环境中运行良好。 但是，随着越来越多的用户从HiveServer迁移到HiveServer2并开始创建大量并发会话，可能会出现问题。 主要问题是Hiveserver2和ZooKeeper之间打开的连接数一直在增加，直到从ZooKeeper服务器端达到连接限制为止。 此时，ZooKeeper开始拒绝新的连接，并且所有依赖ZooKeeper的流都变得不可用。 为了解决这个问题，已经开放了多个Hive JIRA（例如HIVE-4132，HIVE-5853和HIVE-8135等），并且最近通过HIVE-9119对其进行了修复。

让我们仔细看一下Hive中的ZooKeeperHiveLockManager实现，看看它为什么以前引起问题，以及我们如何解决它。

ZooKeeperLockManager使用简单的ZooKeeper API来实现分布式锁。 下面列出了它使用的协议。

希望获得共享锁的客户端应执行以下操作：

1. 调用create（）创建一个路径名称为“ _lockresource_ / lock-shared-”且设置了序列标志的节点。
2. 在不设置监视标志的情况下，在节点上调用getChildren（）。
3. 如果不存在路径名称以“ lock-exclusive-”开头的子代，则客户端将获取该锁并退出。
4. 否则，调用delete（）来删除它们在步骤1中创建的节点，休眠一个预定义的时间段，然后通过执行步骤1直到达到最大重试次数来进行重试。

希望获得排他锁的客户应执行以下操作：

1. 调用create（）创建一个路径名称为“ _lockresource_ / lock-exclusive-”且已设置序列标志的节点。
2. 在不设置监视标志的情况下，在节点上调用getChildren（）。
3. 如果没有序号比在步骤1中创建的节点低的子级，则客户端获取锁并退出。
4. 否则，调用delete（）来删除它们在步骤1中创建的节点，休眠一个预定义的时间段，然后通过执行步骤1直到达到最大重试次数来进行重试。

希望释放锁的客户端应该只删除他们在步骤1中创建的节点。此外，如果所有子节点都已删除，则也要删除父节点。

上面的锁定和解锁协议非常简单明了。 但是，该协议的先前实现未正确使用ZooKeeper客户端。 对于每个Hive查询，都会创建一个新的ZooKeeper客户端实例来获取和释放锁。 这给ZooKeeper服务器处理新的连接带来了很多开销。 另外，在多会话环境中，如果并发查询太多，很容易达到ZooKeeper服务器连接限制。 此外，当用户使用Hue进行Hive查询时，也会发生这种情况。 色相默认情况下不会关闭Hive查询，这意味着为该查询创建的ZooKeeper客户端永远不会关闭。 如果查询量很大，则可以很快达到ZooKeeper连接限制。

我们是否真的需要为每个查询创建一个新的ZooKeeper客户端？ 我们发现没有必要。 从上面的讨论中，我们可以看到HiveServer2使用ZooKeeper客户端与ZooKeeper服务器进行通信，以便能够获取和释放锁。 主要工作量在ZooKeeper服务器端，而不是客户端。 一个HiveKeeper客户端可以由针对HiveServer2服务器的所有查询共享。 使用Singleton ZooKeeper客户端，可以消除处理连接的服务器开销。 Hue用户不再受ZooKeeper连接问题的困扰。

Singleton ZooKeeper客户端能够解决锁定管理问题。 但是，我们仍然需要直接使用ZooKeeper客户端来处理一些额外的事情，例如：

- 初始连接：ZooKeeper客户端和服务器握手需要一些时间。 ZooKeeperHiveLockManager使用的同步方法调用（例如create（），getChildren（），delete（））将在此握手尚未完成时引发异常。 在这种情况下，我们需要一个锁存器来控制ZooKeeper客户端何时开始向服务器发送方法调用。
- 断开连接和故障转移：如果Singleton ZooKeeper客户端失去与服务器的连接，我们需要处理连接重试并将故障转移到集群中的另一台服务器。
- 会话超时：如果发生连接会话超时，则需要关闭单例ZooKeeper客户端并重新创建。

[Apache Curator](https://curator.apache.org/)是开源软件，能够透明地处理所有上述情况。 Curator是Netflix的ZooKeeper库，它提供了简化使用ZooKeeper的高级API-CuratorFramework。 通过在新的ZooKeeperHiveLockManager实现中使用单例CuratorFramework实例，我们不仅解决了ZooKeeper连接问题，而且使代码易于理解和维护。

感谢Hive开源社区将此修复程序包含在Apache Hive 1.1中。 最新的Hive 0.12和Hive 0.13版本以及即将发布的[MapR Distribution的](https://www.mapr.com/) Hive 1.0版本中也包含此修复程序。

## 参考文献：

- ZooKeeper：http： [//zookeeper.apache.org/](https://zookeeper.apache.org/)
- 策展人： [http](https://curator.apache.org/) ： [//curator.apache.org/](https://curator.apache.org/)

## 相关的JIRAS：

- HIVE-4132： [https](https://issues.apache.org/jira/browse/HIVE-4132) ： [//issues.apache.org/jira/browse/HIVE-4132](https://issues.apache.org/jira/browse/HIVE-4132)
- HIVE-5853： [https](https://issues.apache.org/jira/browse/HIVE-5853) ： [//issues.apache.org/jira/browse/HIVE-5853](https://issues.apache.org/jira/browse/HIVE-5853)
- HIVE-8135： [https](https://issues.apache.org/jira/browse/HIVE-8135) ： [//issues.apache.org/jira/browse/HIVE-8135](https://issues.apache.org/jira/browse/HIVE-8135)
- HIVE-9119： [https](https://issues.apache.org/jira/browse/HIVE-9119) ：//issues.apache.org/jira/browse/HIVE-9119





```xml
<property>
  <name>hive.txn.manager</name>
  <value>org.apache.hadoop.hive.ql.lockmgr.DummyTxnManager</value>
  <description>
    Set to org.apache.hadoop.hive.ql.lockmgr.DbTxnManager as part of turning on Hive
    transactions, which also requires appropriate settings for hive.compactor.initiator.on,
    hive.compactor.worker.threads, hive.support.concurrency (true),
    and hive.exec.dynamic.partition.mode (nonstrict).
    The default DummyTxnManager replicates pre-Hive-0.13 behavior and provides
    no transactions.
  </description>
    <property>
  <name>hive.support.concurrency</name>
  <value>true</value>
  <description>
    Whether Hive supports concurrency control or not.
    A ZooKeeper instance must be up and running when using zookeeper Hive lock manager
  </description>
</property>
```

