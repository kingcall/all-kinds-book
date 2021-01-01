[TOC]

## HiveServer 简介

今天我们学习一下Hive 架构中的重要一员HiveServer2或者是HiveServer1，HiveServer2使得其他语言访问Hive 成为了可能，其他语言通过连接HiveServer2服务提供的接口进而访问Hive,HiveServer2还引入了一个客户端，那就是大名鼎鼎的BeeLine,BeeLine 是一个通过JDBC 访问Hive的shell 接口

其实我们在前面讲[Hive的架构设计](https://blog.csdn.net/king14bhhb/article/details/111769279) 的时候提到过，Hive 提供的另外一个shell 客户端，也就是我们常用的hive 命令的客户端它的设计是直接启动了一个`org.apache.hadoop.hive.cli.CliDriver`的进程，这个进程其实主要包含了两块内容一个是提供给我们交互的cli ，另外一个就是我们的Driver 驱动引擎，这样的设计导致如果我们有多个客户端的情况下，我们就需要有多个Driver，但是我们通过HiveServer2连接的时候我们就可以共享`Driver`,一方面可以简化客户端的设计降低资源损耗，另外一方面还能降低对MetaStore 的压力，减少连接的个数。



![image-20201227123352163](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/27/12:33:52-image-20201227123352163.png)



### 1. HiveServer1

HiveServer是一种可选服务，允许远程客户端可以使用各种编程语言向Hive提交请求并检索结果。**HiveServer是建立在[Apache ThriftTM](http://thrift.apache.org/)之上的，因此有时会被称为Thrift Server**，这可能会导致混乱，因为新服务HiveServer2也是建立在Thrift之上的．自从引入HiveServer2后，HiveServer也被称为HiveServer1。

HiveServer1无法处理来自多个客户端的并发请求，这实际上是HiveServer导出的Thrift接口所施加的限制，也不能通过修改HiveServer源代码来解决。HiveServer2对HiveServer1进行了重写，来解决这些问题，从Hive 0.11.0版本开始,建议使用HiveServer2。

### **2. HiveServer2**

#### 2.1 引入

HiveServer2(HS2)是一种能使客户端执行Hive查询的服务。 HiveServer2是HiveServer1的改进版，HiveServer1已经被废弃。HiveServer2可以支持多客户端并发和身份认证。旨在为开放API客户端（如JDBC和ODBC）提供更好的支持。

HiveServer2单进程运行，提供组合服务，包括基于Thrift的Hive服务(TCP或HTTP)和用于Web UI的Jetty Web服务器。

#### 2.2 架构

基于Thrift的Hive服务是HiveServer2的核心，负责维护Hive查询（例如，从Beeline）。Thrift是构建跨平台服务的RPC框架。其堆栈由4层组成：server，Transport，Protocol和处理器。可以在 [https://thrift.apache.org/docs/concepts](https://link.zhihu.com/?target=https%3A//thrift.apache.org/docs/concepts) 找到有关分层的更多详细信息。

##### **2.2.1 Server**

HiveServer2在TCP模式下使用TThreadPoolServer(来自Thrift)，在HTTP模式下使用Jetty Server。

TThreadPoolServer为每个TCP连接分配一个工作线程。即使连接处于空闲状态，每个线程也始终与连接相关联。因此，由于大量并发连接产生大量线程，从而导致潜在的性能问题。在将来，HiveServer2可能切换到TCP模式下的另一个不同类型的Server上，例如TThreadedSelectorServer。

##### **2.2.2 Transport**

如果客户端和服务器之间需要代理(例如，为了负载均衡或出于安全原因)，则需要HTTP模式。这就是为什么它与TCP模式被同样支持的原因。可以通过Hive配置属性`hive.server2.transport.mode`指定Thrift服务的传输模式。

##### **2.2.3 Protocol**

协议责序列化和反序列化。HiveServer2目前正在使用`TBinaryProtocol`作为Thrift的协议进行序列化。 在未来，可以更多考虑其他协议，如TCompactProtocol，可以考虑更多的性能评估。

##### **2.2.4 处理器**

处理流程是处理请求的应用程序逻辑。例如，`ThriftCLIService.ExecuteStatement()`方法实现了编译和执行Hive查询的逻辑。

#### 2.3 依赖

- Metastore metastore可以配置为嵌入式（与HiveServer2相同的过程）或远程服务器（也是基于Thrift的服务）。 HS2与查询编译所需的元数据相关。
- Hadoop cluster HiveServer2准备了各种执行引擎（MapReduce/Tez/Spark）的物理执行计划，并将作业提交到Hadoop集群执行。

### 3. HiveServer 的使用方式

#### 1. JDBC Client

推荐使用JDBC驱动程序让客户端与HiveServer2进行交互。请注意，有一些用例（例如，Hadoop Hue），直接使用Thrift客户端，而没有使用JDBC。 以下是进行第一次查询所涉及的一系列API调用：

- JDBC客户端（例如，Beeline）通过初始化传输连接(例如，TCP连接)，再调用OpenSession API来获取SessionHandle来创建HiveConnection。 会话是从服务器端创建的。
- 执行HiveStatement（遵循JDBC标准），并且Thrift客户端调用ExecuteStatement API。 在API调用中，SessionHandle信息与查询信息一起传递给服务器。
- HiveServer2服务器接收请求，并让驱动程序（CommandProcessor）进行查询解析和编译。该驱动程序启动后台工作，将与Hadoop交互，然后立即向客户端返回响应。这是ExecuteStatement API的异步设计。响应包含从服务器端创建的OperationHandle。
- 客户端使用OperationHandle与HiveServer2交互以轮询查询执行的状态。

#### 2. beeline

  beeline方式连接：beeline -u jdbc:hive2//localhost:10000/default -n root -p 123456  或者java client方式连接

  备注：连接Hive JDBC URL：jdbc:hive://192.168.6.116:10000/default   （Hive默认端口：10000 默认数据库名：default）

#### 3. Thrift客户端访问

因为HiveServer2是基于Thrift实现的，所以我们也可以使用 Thrift客户端来访问Hive服务,但是推荐使用JDBC的方式访问，主要是考虑到代码的性能和统一访问方式，关于更多细节请看[Hive的其他语言调用](https://blog.csdn.net/king14bhhb/article/details/111770561)

**Python 操作**

```python
import sys
 
from hive import ThriftHive
from hive.ttypes import HiveServerException
from thrift import Thrift
from thrift.transport import TSocket
from thrift.transport import TTransport
from thrift.protocol import TBinaryProtocol
 
try:
    transport = TSocket.TSocket('localhost', 10000)
    transport = TTransport.TBufferedTransport(transport)
    protocol = TBinaryProtocol.TBinaryProtocol(transport)
    
    client = ThriftHive.Client(protocol)
    transport.open()
    
    client.execute("CREATE TABLE r(a STRING, b INT, c DOUBLE)")
    client.execute("LOAD TABLE LOCAL INPATH '/path' INTO TABLE r")
    client.execute("SELECT * FROM r")
    while (1):
      row = client.fetchOne()
      if (row == None):
        break
      print row
    client.execute("SELECT * FROM r")
    print client.fetchAll()
    
    transport.close()
except Thrift.TException, tx:
    print '%s' % (tx.message)
```



**Php操作**

```java

<?php
// set THRIFT_ROOT to php directory of the hive distribution
$GLOBALS['THRIFT_ROOT'] = '/lib/php/';
// load the required files for connecting to Hive
require_once $GLOBALS['THRIFT_ROOT'] . 'packages/hive_service/ThriftHive.php';
require_once $GLOBALS['THRIFT_ROOT'] . 'transport/TSocket.php';
require_once $GLOBALS['THRIFT_ROOT'] . 'protocol/TBinaryProtocol.php';
// Set up the transport/protocol/client
$transport = new TSocket('localhost', 10000);
$protocol = new TBinaryProtocol($transport);
$client = new ThriftHiveClient($protocol);
$transport->open();
 
// run queries, metadata calls etc
$client->execute('SELECT * from src');
var_dump($client->fetchAll());
$transport->close();
```



## 安装 HiveServer2

### 最小配置

```properties
<!-- 这是hiveserver2 -->
<property>
         <name>hive.server2.thrift.port</name>
       <value>10000</value>
</property>
<property>
 		<name>hive.server2.thrift.bind.host</name>
 		<value>localhost</value>
</property>
```

这个最小配置我们在前面的[数仓工具—Hive安装](https://blog.csdn.net/king14bhhb/article/details/111568313) 中已经介绍过了，所谓最小配置就是启动不报错，可以用的状态

### 启动

可以使用`hive --service hiveserver2` 或者 `hiveserver2` 来启动服务，但是我们遇到端口占用的情况，因为我是本地电脑，所以端口冲突的概率比较大

![image-20201223082428817](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/08:24:30-image-20201223082428817.png)

这里因为10000 端口被占，所以我们只需要kill 掉相关的服务(你也可选择为`hiveserver2`重新选择一个端口)

![image-20201223082534904](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/08:25:35-image-20201223082534904.png)

接下来我们直接启动，发现启动成功了

```
2020-12-23 09:00:22,074 INFO  [main] server.AbstractConnector (AbstractConnector.java:doStart(278)) - Started ServerConnector@7da34b26{HTTP/1.1,[http/1.1]}{0.0.0.0:10002}
2020-12-23 09:00:22,074 INFO  [main] server.Server (Server.java:doStart(414)) - Started @64032ms
2020-12-23 09:00:22,075 INFO  [main] http.HttpServer (HttpServer.java:start(255)) - Started HttpServer[hiveserver2] on port 10002
2020-12-23 09:00:22,075 INFO  [main] server.HiveServer2 (HiveServer2.java:start(735)) - Web UI has started on port 10002
2020-12-23 09:00:22,075 INFO  [main] server.HiveServer2 (HiveServer2.java:start(743)) - HS2 interactive HA not enabled. Starting tez sessions..
2020-12-23 09:00:22,075 INFO  [main] server.HiveServer2 (HiveServer2.java:startOrReconnectTezSessions(802)) - Starting/Reconnecting tez sessions..
2020-12-23 09:00:22,075 INFO  [main] server.HiveServer2 (HiveServer2.java:initAndStartTezSessionPoolManager(831)) - Initializing tez session pool manager
2020-12-23 09:00:22,075 INFO  [main] server.HiveServer2 (HiveServer2.java:initAndStartTezSessionPoolManager(840)) - Tez session pool manager initialized.
2020-12-23 09:00:22,075 INFO  [main] server.HiveServer2 (HiveServer2.java:initAndStartWorkloadManager(858)) - Workload management is not enabled.
```



### Web UI for HiveServer2

从启动日志里，我们看到了一个信息，` Web UI has started on port 10002` 我们尝试打开这个这个地址

![image-20201223083108167](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/08:31:08-image-20201223083108167.png)

你可以将这个理解为HiveServer2的web 端的监控,然后你在上面只能看到一个session,其实就是连接上来的客户端，我们再开一个看看

![image-20201223083640524](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/08:36:41-image-20201223083640524.png)

可以看到现在活动状态色sessions 已经有两个了

在这个页面上你还是可以拿到很多监控信息的，所以就看你如何去使用了



### beeline for HiveServer2

前面我们介绍到了beeline是HiveServer2 新引入的客户端，接下来我们简单演示一下beeline，因为我们有单独的章节去介绍beeline

![image-20201223082820562](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/08:28:21-image-20201223082820562.png)

你会看到如下的日志信息，其实这个信息挺重要的，你可以有何印象，例如`TRANSACTION_REPEATABLE_READ`

```
Connecting to jdbc:hive2://localhost:10000
Connected to: Apache Hive (version 3.1.2)
Driver: Hive JDBC (version 3.1.2)
Transaction isolation: TRANSACTION_REPEATABLE_READ
Beeline version 3.1.2 by Apache Hive
```

你也可以进入beeline 之后再去进行连接

```
// 也可以在连接时指定数据库 !connect jdbc:hive2://localhost:10000/ods　　（ods 是数据库名)
// 还可以指定用户名和密码  !connect jdbc:hive2://localhost:10000 root  www1234
```

![image-20201223084107865](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/08:41:08-image-20201223084107865.png)

![image-20201223084031347](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/08:40:31-image-20201223084031347.png)

然后你就可以看到我新连接上去的用户,如果你去看日志，你会发现匿名用户连接上去是有个警告信息的，但是你最终还是可连接上去的

```
upsMapping.java:getUnixGroups(210)) - unable to return groups for user anonymous
PartialGroupNameException The user name 'anonymous' is not found. id: anonymous: no such user
id: anonymous: no such user

	at org.apache.hadoop.security.ShellBasedUnixGroupsMapping.resolvePartialGroupNames(ShellBasedUnixGroupsMapping.java:294)
	at org.apache.hadoop.security.ShellBasedUnixGroupsMapping.getUnixGroups(ShellBasedUnixGroupsMapping.java:207)
	at org.apache.hadoop.security.ShellBasedUnixGroupsMapping.getGroups(ShellBasedUnixGroupsMapping.java:97)
	at org.apache.hadoop.security.JniBasedUnixGroupsMappingWithFallback.getGroups(JniBasedUnixGroupsMappingWithFallback.java:51)
	at org.apache.hadoop.security.Groups$GroupCacheLoader.fetchGroupList(Groups.java:387)
	at org.apache.hadoop.security.Groups$GroupCacheLoader.load(Groups.java:321)
	at org.apache.hadoop.security.Groups$GroupCacheLoader.load(Groups.java:270)
	at com.google.common.cache.LocalCache$LoadingValueReference.loadFuture(LocalCache.java:3528)
	at com.google.common.cache.LocalCache$Segment.loadSync(LocalCache.java:2277)
	at com.google.common.cache.LocalCache$Segment.lockedGetOrLoad(LocalCache.java:2154)
	at com.google.common.cache.LocalCache$Segment.get(LocalCache.java:2044)
	at com.google.common.cache.LocalCache.get(LocalCache.java:3952)
	at com.google.common.cache.LocalCache.getOrLoad(LocalCache.java:3974)
	at com.google.common.cache.LocalCache$LocalLoadingCache.get(LocalCache.java:4958)
	at org.apache.hadoop.security.Groups.getGroups(Groups.java:228)
	at org.apache.hadoop.security.UserGroupInformation.getGroups(UserGroupInformation.java:1588)
	at org.apache.hadoop.security.UserGroupInformation.getGroupNames(UserGroupInformation.java:1576)
	at org.apache.hadoop.hive.metastore.HiveMetaStoreClient.open(HiveMetaStoreClient.java:534)
	at org.apache.hadoop.hive.metastore.HiveMetaStoreClient.<init>(HiveMetaStoreClient.java:224)
```

所以这个时候其实你可以猜到我们这里其实应该是没有做限制的，因为目前我们用的都还是搭建hive 时候用的最小配置，只是为了让它跑起来,因为匿名用户可以进来，所以其实我们指定任意用户名都可以进来，就像下面的`kingcall`

![image-20201223084959662](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/08:50:00-image-20201223084959662.png)

但是你依然可以看到告警信息，**这是因为如果没有专门配置用户名和密码，他们分别是 关系型数据库，也就是存放元数据的数据库的 用户名和密码**

### 二次配置

这里我们为其配置特定的用户名和密码，再次登录的时候它就需要我们输入用户名和密码了

```xml
<property>
  <name>beeline.hs2.connection.user</name>
  <value>hive-beeline</value>
</property>
<property>
  <name>beeline.hs2.connection.password</name>
  <value>www1234</value>
</property>
```

![image-20201223090239918](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/09:02:40-image-20201223090239918.png)

## 总结

1. HiveServer2是一种可选服务，允许远程客户端可以**使用各种编程语言**向Hive提交请求并检索结果,HiveServer2对HiveServer1进行了重写解决了HiveServer1**不支持多客户端连接的问题**
2. **HiveServer2支持以JDBC 的方式访问，也支持直接使用Thrift客户端访问**
3. HiveServer2如果没有专门配置用户名和密码，可以使用关系型数据库，也就是存放元数据的数据库的 用户名和密码，也支持匿名用户访问
4. HiveServer2 提供了客户端beeline 和 Web UI for HiveServer2

