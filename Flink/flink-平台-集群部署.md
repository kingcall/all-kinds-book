[toc]
# 《集群部署 和项目部署》
   - flink作为大数据生态圈的一员，它和Hadoop的hdfs是兼容的。
   - 一般将namenode和jobmanager部署到一起，将datanode和taskmanager部署到一起。
   - flink也能照顾到数据的本地行，移动计算而不是移动数据
      ![](image/flink_cluster.png)
      ![](image/flink_hdfs.png)
## 进程和资源划分
  -  Flink运行时主要角色有两个：JobManager和TaskManager，无论是standalone集群，flink on yarn都是要启动这两个角色。
  - JobManager主要是负责接受客户端的job，调度job，协调checkpoint等。  
  - TaskManager执行具体的Task。
  - Client这个角色主要是为job提交做些准备工作，比如构建jobgraph提交到jobmanager，提交完了可以立即退出，当然也可以用client来监控进度。 
### Task 的定义
#### spark 中的Task 
   - RDD中的一个分区对应一个task，task是单个分区上最小的处理流程单元。被送到某个Executor上的工作单元，和hadoopMR中的MapTask和ReduceTask概念一样，是运行Application的基本单位，多个Task组成一个Stage
#### Flink 中的Task 
   - 每个算子的一个并行度实例就是一个subtask-在这里为了区分暂时叫做substask。
   - 那么，带来很多问题，由于flink的taskmanager运行task的时候是每个task采用一个单独的线程，这就会带来很多线程切换开销，进而影响吞吐量。
   - 为了减轻这种情况，flink进行了优化，也即对subtask进行链式操作，链式操作结束之后得到的task，再作为一个调度执行单元，放到一个线程里执行
### slot
#### 任务槽
   - TaskManager为了对资源进行隔离和增加允许的task数，引入了slot的概念，这个slot对资源的隔离仅仅是对内存进行隔离，策略是均分 
   - TaskManager最多能同时并发执行的任务是可以控制的，那就是slot的数量。
   - slot有独占的内存空间，这样在一个TaskManager中可以运行多个不同的作业，作业之间不受影响。
   - slot之间可以共享JVM资源, 可以共享Dataset和数据结构，也可以通过多路复用（Multiplexing）共享TCP连接和心跳消息（Heatbeat Message）
   - 每个 TaskManager 有一个slot，也就意味着每个task运行在独立的 JVM 中。每个 TaskManager 有多个slot的话，也就是说多个task运行在同一个JVM中。
     而在同一个JVM进程中的task，可以共享TCP连接（基于多路复用）和心跳消息，可以减少数据的网络传输，也能共享一些数据结构，一定程度上减少了每个task的消耗。
   - 每个TaskManager是一个JVM进程，会在不同的线程中执行一个或多个operator subtask。为了控制单个TaskManager所能接收的任务数量，每个TaskManager会包含一组Task槽（至少会有一个）。
   - 一个TaskManager JVM进程会有一个或多个Task Slot（个数一般与cpu core的个数相等），每个Task Slot能分配到这个JVM中的一部分资源（内存）；
#### 共享槽
   - flink默认允许同一个job下的subtask可以共享slot
        - 这样可以使得同一个slot运行整个job的流水线（pipleline）—— 每个slot可以接受单个task，也可以接受多个连续task组成的pipeline
        - 如果这样不同Operator 之间的数据传输就是缓存，而不是网络
   - flink 不允许不同作业的任务共享一个slot
#### 优势
   - 只需计算Job中最高并行度（parallelism）的task slot,只要这个满足，其他的job也都能满足。 
   - Flink 集群所需的taskslots数与job中最高的并行度一致  
#### slot和parallelism    
   - slot是指taskmanager的并发执行能力
   - parallelism是指taskmanager实际使用的并发能力,slot是静态的概念，是指taskmanager具有的并发执行能力，
      运行程序默认的并行度为1，9个TaskSlot只用了1个，有8个空闲。设置合适的并行度才能提高效率。

#### 约束调度关系
   - ColocationGroup  决定哪些任务必须在同一个slot 上运行
   - SlotSharingGroup 决定哪些任务可以在一个slot 上运行
   - Flink 根据集群资源的使用情况最优地调度任务执行  
### JobManager
   - JobManager负责接收 flink 的作业，调度 task，收集 job 的状态、管理 TaskManagers。被实现为一个 akka actor。
   - 将 Client 提交的JobGraph 转换成 ExecutionGraph 
#### 组件
   - BlobServer 是一个用来管理二进制大文件的服务，比如保存用户上传的jar文件，该服务会将其写到磁盘上。还有一些相关的类，如BlobCache，用于TaskManager向JobManager下载用户的jar文件
   - InstanceManager 用来管理当前存活的TaskManager的组件，记录了TaskManager的心跳信息等
   - CompletedCheckpointStore 用于保存已完成的checkpoint相关信息，持久化到内存中或者zookeeper上
   - MemoryArchivist 保存了已经提交到flink的作业的相关信息，如JobGraph等      
### TaskManager
   - TaskManager是flink中资源管理的基本组件，是所有执行任务的基本容器，提供了内存管理、IO管理、通信管理等一系列功能，本节对各个模块进行简要介绍
#### 组件
   - MemoryManager flink并没有把所有内存的管理都委托给JVM，因为JVM普遍存在着存储对象密度低、大内存时GC对系统影响大等问题。所以flink自己抽象了一套内存管理机制
   - IOManager flink通过IOManager管理磁盘IO的过程，提供了同步和异步两种写模式，又进一步区分了block、buffer和bulk三种读写方式。
     IOManager提供了两种方式枚举磁盘文件，一种是直接遍历文件夹下所有文件，另一种是计数器方式，对每个文件名以递增顺序访问。
     在底层，flink将文件IO抽象为FileIOChannle，封装了底层实现。
### client
   - client属于flink架构的一部分，但不属于flink集群。它的工作是连接user和cluster.  
        a.client能够将user提交的application分析成Dataflow提交给JM.JM会分配给TM做具体的执行工作。
            在提交完Dataflow可以关闭，也可以不关闭.
        b.client不关闭的话还可以接受cluster处理进度报告，以便user能跟着任务的运行情况  
## 任务调度
### 并行度
 - 每个Opertor 的实例数为并行度，并且任意两个Operator 之间的并行度是独立的；
   每个Opertor称为一个任务，每个Opertor 的实例称为子任务（subtask）
 - flink架构是分布式的，也就决定了程序（Progrram）和数据流（Dataflows）也是分别式的。
 - Dataflow也是一个分布式概念，它的Stream被查分成Stream-Partition,Operator被查分成subtask.
 - Stream-Partition本质就是data-partition,subtask本质是thread.
 - 这些subtask(thread)相互独立，被分配到不同的机器上并行执行，甚至是不同的container中并行执行。
 - 也就是说在每一个数据分片上运行一个线程这些独立的线程能够并行的处理数据。所以，Stream的分区数和Operator的并行度是一致的。只不过Stream-Partition
   是描述数据被分片的情况，Operator-subtask是描述线程的并行情况。
 - 一个Operator被查分成subtask的数量就是并行度（parallelism），它决定这程序并发执行的线程个数。设置合适的并行度，能够使任务在不同的机器上并行执行，能提高程序的运行效率。

### 数据传输模式
#### Forwarding
  Forwarding模式是指Stream-Partition之间一对一(One-to-One)传输。子stream保留父stream的分区个数和元素的顺序。Source向map传输stream-partition就在这种情况，分区个数，元素顺序都能保持不变，这里可以进行优化。
  可以把source和map做成一个TaskChain,用一个thread去执行一个source-subtask和map-subtask.原本4个thread处理的任务，优化后2个thread就能完成了，因为减少了不必要的thread开销，效率还能提升。
  - 可以将Forwarding模式下的多个subtask做成一个subtask-chain,将一个thread对应一个subtask优化为一个thread对应一个subtask-chain中的多个subtask。可提高总体吞吐量（throughput）并降低延迟（latency）。
  - 如果说stream-partition对数据分区是为了通过提高并发度，来提高程序的运行效率。那么subtask-chain就是在程序的运行过程中合并不必要的thread来提高程序的运行效率。
#### Redistributing
 Redistributing模式是指Stream-Partition之间是多对多的传输。stream转化过程中partition之间进行了shuffer操作,这会把分区个数和元素顺序全部打乱，可能会牵涉到数据的夸节点传输。
 因为数据可能夸节点传输，无法确定应该在哪个节点上启动一个thread去处理在两个节点上的数据，因此无法将Redistributing模式下的task做成一个task-chain。Map-KeyBy/Window和KeyBy/Window-sink直接就是Redistributing模式。  

1.client将program解析成dataflow,并将dataflow的描述信息JobGraph提交给JobManager。
JobGraph包含Operator(JobVertex),IntermediateResult(IntermediateDataSet),并行度,执行代码,附加的库等信息。

2.JobManager将JobGraph并行化处理成ExecutionGraph。
Operator（JobVertex）处理成包含多个Subtask(ExecutionVertex)的ExecutionJobVertex。
IntermediateResult(IntermediateDataSet)并行化成含多个IntermediateResultPartition的IntermediateResult.
也就是
任务并行化： JobVertex->ExecutionJobVertex(含多个ExecutionVertex)
数据并行化： IntermediateResult->IntermediateResult(含多个IntermediateResultPartition)

## 部署模式
   - JM和TM有多种部署方式，可以选择使用裸机部署，使用container部署，使用yarn部署。
      只要JM和TM能通信即可，这样JM就能下发任务到TM,TM也能执行任务并上报TM.
### 单节点运行
   - 开箱即用的配置将使用您的默认Java安装.您可以手动设置环境变量JAVA_HOME或配置项env.java.home中conf/flink-conf.yaml，如果你想手动覆盖Java运行时使用.
### Flink on Yarn
   -  YARN上的Flink将覆盖以下配置参数jobmanager.rpc.address（因为JobManager总是在不同的机器上分配）
      taskmanager.tmp.dirs（我们使用YARN给出的tmp目录）以及parallelism.default是否已指定插槽数。 
#### yarn session
   - 如果您不想让Flink YARN客户端一直运行，也可以启动分离的 YARN会话。该参数称为-d或--detached。
     在这种情况下，Flink YARN客户端将仅向群集提交Flink，然后自行关闭。请注意，在这种情况下，无法使用Flink停止YARN会话。 
   - 可用参数配置
       ```text
        Usage:
           Required
             -n,--container <arg>   Number of YARN container to allocate (=Number of Task Managers)
           Optional
             -D <arg>                        Dynamic properties
             -d,--detached                   Start detached
             -jm,--jobManagerMemory <arg>    Memory for JobManager Container with optional unit (default: MB)
             -nm,--name                      Set a custom name for the application on YARN
             -q,--query                      Display available YARN resources (memory, cores)
             -qu,--queue <arg>               Specify YARN queue.
             -s,--slots <arg>                Number of slots per TaskManager
             -tm,--taskManagerMemory <arg>   Memory per TaskManager Container with optional unit (default: MB)
             -z,--zookeeperNamespace <arg>   Namespace to create the Zookeeper sub-paths for HA mode
       ```
   - 其实就是相当于Flink 集群，和你用的standalone 集群相似
   - 日志(你可以看到日志提示是任务发现了JobMannager)
       ```text
         accessing YARN.
         2019-04-17 07:28:36,616 INFO  org.apache.flink.yarn.AbstractYarnClusterDescriptor   - Found application JobManager host name 'dev130.creativearts.cn' and port '38843' from supplied application id 'application_1554262753394_0044'
         Starting execution of program
       ```
   - 进程名称其实不叫 Jobmanager 而是 StandaloneSessionClusterEntrypoint，但是Taskmanager 就是叫 TaskManagerRunner
     ![](image/cluster/集群部署进程名称.PNG)
   - 在 flink中1.5-1.7 的版本中不论是yarn-session(n) 还是yarn 中(yn) 都是没有生效的 ，而是借助 -p 参数 
##### 常用命令
   -  ./bin/yarn-session.sh -n 4 -jm 1024m -tm 4096m
   -  会话启动后，您可以使用该./bin/flink工具将作业提交到群集。
   -  发出以下命令以分配10个TaskManager，每个管理器具有8 GB内存和32个处理插槽：
   - ./bin/yarn-session.sh -n 10 -tm 8192 -s 32
   - ./bin/yarn-session.sh -id application_1463870264508_0029 附加到现有会话
   - ./bin/flink run ./com.kingcallj.examples/batch/WordCount.jar 提交作业到会话
   - ./bin/flink run -m myJMHost:8081 com.kingcallj.examples/batch/WordCount.jar  指定 JobManager 运行
##### sesssion 例子
   - flink run ${FLINK_HOME}/examples/batch/WordCount.jar
      ![](image/yarn/yarn-session.jpg)
   - 
##### 总结
1. flink standalone
	${FLINK_HOME}/bin/flink run  -m qingcheng11:6123  simple-flink-1.0-SNAPSHOT.jar
2. flink on yarn
    1. 临时session
        $FLINK_HOME/bin/flink run -m yarn-cluster -yn 3  /bigdata/software/simple-flink.jar
    2. 长期session
	    - 后台启动yarn-session
			$FLINK_HOME/bin/yarn-session.sh -n 3 -s 3 -d
		- 运行jar包
			$FLINK_HOME/bin/flink run /bigdata/software/simple-flink.jar


#### yarn-cluster
   首先flink-client将一个job提交到yarn。yarn为这个application启动一个临时flink集群，application运行结束后后,yarn关闭临时flink集群
   - 上面的文档描述了如何在Hadoop YARN环境中启动Flink集群。也可以仅在执行单个作业时在YARN中启动Flink。
   - ./bin/flink run -m yarn-cluster -yn 4 -yjm 1024m -ytm 4096m ./com.kingcallj.examples/batch/WordCount.jar
   - 启动命名参数
        - -m yarn-cluster (指定创建新的Flink 集群)
        - -p,--parallelism <parallelism>       The parallelism with which to run the program. Optional flag to override the default value specified in the configuration.
        - q 禁用日志输出
        - -yn,--yarncontainer <arg>            Number of YARN container to allocate(=Number of Task Managers) (两个TaskManager)
        - -ynm,--yarnname <arg>                Set a custom name for the application  on YARN
        - -yq,--yarnquery                      Display available YARN resources(memory, cores)
        - -yqu,--yarnqueue <arg>               Specify YARN queue.
        - -ys,--yarnslots <arg>                Number of slots per TaskManager
        - -ytm,--yarntaskManagerMemory <arg>   Memory per TaskManager Container with optional unit (default: MB)
        - -yjm,--yarnjobManagerMemory <arg>    Memory for JobManager Container with optional unit (default: MB)
        - -yid --yarnapplicationId <arg>       Attach to running YARN session
### HA
#### 独立群集高可用性(往往是Flink 的原生集群)
   - 当Flink程序运行时，如果jobmanager崩溃，那么整个程序都会失败,并且服务提交新的任务。为了防止jobmanager的单点故障，
      借助于zookeeper的协调机制，可以实现jobmanager的HA配置—-1主（leader）多从（standby）。 
   - 独立集群的JobManager高可用性的一般概念是，任何时候都有一个Master JobManager和多个Slave JobManagers，以便在Leader失败时接管领导。
      这保证了没有单点故障，一旦Slave JobManager取得领导，程序就可以取得进展。Slave 和主JobManager实例之间没有明确的区别。
   - 要启用JobManager高可用性，您必须将高可用性模式设置为zookeeper，配置ZooKeeper quorum并设置包含所有JobManagers主机及其Web UI端口的主服务器文件。
   - Flink利用ZooKeeper在所有正在运行的JobManager实例之间进行分布式协调。ZooKeeper是Flink的独立服务，通过Leader选举和轻量级一致状态存储提供高度可靠的分布式协调
   - 由于flink的HA模式下的state backend在要依赖hdfs，因此要先配置并启动Hadoop集群
##### 配置
   - 要启动HA群集，请在以下位置配置主文件conf/masters
   ````text
    jobManagerAddress1：webUIPort1 
    jobManagerAddressX：webUIPortX
   ````
  - 配置文件（flink-conf.yaml）
  ```text
    high-availability: zookeeper
    high-availability.zookeeper.quorum: dev110:2181,dev120:2181,dev130:2181
    high-availability.storageDir: hdfs://dev110:8020/flink/ha/
    high-availability.zookeeper.path.root: /flink
    high-availability.cluster-id: /flinkCluster
    high-availability.zookeeper.client.acl: open
  ```
#### Yarn 集群高可用
   - 在运行高可用性YARN群集时，我们不会运行多个JobManager（ApplicationMaster）实例，而只会运行一个
      由YARN在失败时重新启动。确切的行为取决于您使用的特定YARN版本。
##### 配置
   - yarn
       ```text
       您必须配置为尝试应用Masters 的最大数量的 YARN的设置yarn-site.xml：
       <property>
         <name>yarn.resourcemanager.am.max-attempts</name>
         <value>4</value>
         <description>
           The maximum number of application master execution attempts.在AM 挂掉之后 RM 将其重新启动重试的次数
         </description>
       </property>
       ```
   - Flink
       ```text
       除HA配置（参见上文）外，您还必须配置最大尝试次数conf/flink-conf.yaml：
       yarn.application-attempts：10
       ```
### Blink 编译部署
   git clone -b 分支名 仓库地址
   -  git clone -b blink https://github.com/apache/flink.git 
   - mvn clean install -Dmaven.test.skip -Dcheckstyle.skip -Dlicense.skip=true -Drat.ignoreErrors=true
## 安全认证
## 常用命令
   - flink cancel <jobID>
   - flink cancel -s [targetDirectory] <jobID> 
        - 如果未配置保存点目录，则需作业。
        - flink cancel -s 099b9f88d5d850998d2a7d06b1fa0a9d -yid application_1554262753394_0049  
        - flink run -m yarn-cluster -yn 1 -yjm 1024 -ytm 2048 -ys 1 -ynm historyData -s hdfs://dev110:8020/flink/checkpoints/savepoint-099b9f-73bbf0896394 -c com.zykj.com.kingcallj.examples.stream.kafka.ReadingKafka /data/flink/zyflink.jar
   - flink stop <jobID> 
        - 停止任务（仅限流处理工作）
        - 是一种更优雅的方式来停止正在运行的流处理作业
        - Stop仅适用于使用实现StoppableFunction接口的源的作业
        - 当用户请求停止作业时，所有源都将接收stop()方法调用。该工作将继续运行，直到所有资源正常关闭。这允许作业完成处理所有飞行数据。 
   - flink savepoint <jobId> [savepointDirectory] 
        - 这将触发具有ID的作业的保存点jobId，并返回创建的保存点的路径。您需要此路径来还原和部署保存点。
        - 您可以选择指定目标文件系统目录以存储保存点。该目录需要可由JobManager访问。
        - 如果未指定目标目录，则需要配置默认目录。否则，触发保存点将失败。
   - flink savepoint <jobId> [savepointDirectory] -yid <yarnAppId>
        - 这将触发具有ID jobId和YARN应用程序ID 的作业的保存点yarnAppId，并返回创建的保存点的路径。 
        - flink savepoint 099b9f88d5d850998d2a7d06b1fa0a9d -yid application_1554262753394_0049  
        - flink run -m yarn-cluster -yn 1 -yjm 1024 -ytm 2048 -ys 1 -ynm historyData -s hdfs://dev110:8020/flink/checkpoints/savepoint-099b9f-73bbf0896394 -c com.zykj.com.kingcallj.examples.stream.kafka.ReadingKafka /data/flink/zyflink.jar
   - flink run -s <savepointPath>  
        - run命令有一个保存点标志来提交作业，该作业从保存点恢复其状态。
   - flink run -s <savepointPath> -n 
        - 如果您的程序删除了属于保存点的 算子，这将非常有用。
   - 其他
   ```text
    配置保存点
    ./bin/flink savepoint -d <savepointPath>
    在给定路径处理保存点。savepoint trigger命令返回保存点路径。
    如果使用自定义状态实例（例如自定义还原状态或RocksDB状态），则必须指定触发保存点的程序JAR的路径，以便使用用户代码类加载器处理保存点：
    ./bin/flink savepoint -d <savepointPath> -j <jarFile>
    否则，你会遇到一个ClassNotFoundException。
   ```
## 概念
   - TaskManager 集群的slave,和 Jobmanager 对应 
## 应用升级
   -  ./bin/flink cancel -s [pathToSavepoint] <jobID>
   - ./bin/flink run -d -s [pathToSavepoint] ~/application.jar 
      给定从应用程序获取的保存点，可以从该保存点启动相同或兼容的应用程序
      启动应用程序的算子在获取保存点时使用原始应用程序的算子状态（即从中获取保存点的应用程序）进行初始化。启动的应用程序从这一点开始继续处理。
## 任务资源规划
## 日志
   log4j-cli.properties：由Flink命令行客户端（例如flink run）使用（不是在集群上执行的代码）
   log4j-yarn-session.properties：启动YARN会话时由Flink命令行客户端使用（yarn-session.sh）
   log4j.properties：JobManager / Taskmanager日志（独立和YARN）
## 压测
- Flink使用分布式阻塞队列来作为有界缓冲区
- 如同Java里通用的阻塞队列跟处理线程进行连接一样，一旦队列达到容量上限， 一个相对较慢的接受者将拖慢发送者。 
## 交互式使用
### 远程连接
   - start-scala-shell.sh remote dev10 6123(参数可以看stand-alone 集群的web页面)
### yarn
   - yarn-session
        - 先启动一个yarn-session
        - start-scala-shell.sh yarn
   - yarn-cluster
        - start-scala-shell.sh yarn -n 2
### 本地连接
- start-scala-shell.sh local
![image-20210218190652212](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210218190652212.png)
**备注** vim .bashrc 然后添加 alias flink-shell='start-scala-shell.sh remote dev10 6123'

#### 使用
```text
flink run -p 8 ${FLINK_HOME}/examples/batch/WordCount.jar \
--input  hdfs://dev110:8020/input/flink/README.txt \
--output hdfs://dev110:8020/output/flink/readme_result
```
- start-scala-shell.sh local
- start-scala-shell.sh yarn

## 项目部署
### 核心Jar包的配置
   - Flink-table等jar 包不要打在项目里，而应该配置在集群上 
#### 默认依赖在Lib 目录下(启动的时候会加载)
   ![](image/cluster/Flink lib jar.png)
   - 可以查看这个jar(Fatjar)到底提供了那些东西——其实可以看出核心的一切东西都提供了

    ![](image/cluster/Flinkdistjar.PNG)
#### 可选依赖在 opt 目录下,启动的时候不会加载
   ![](image/cluster/Flinkoptjar.png)
   - 在需要使用的时候移到lib 目录下即可
### 任务的取消
   - stop 优雅的停止任务，前提是任务的source 和 sink 都实现了 stop 接口
   - cancel
### 任务提交的两种模式
   - attach
   - detached 
## 指标监控
   - Gauge —— 最简单的度量指标，只是简单的返回一个值，比如返回一个队列中当前元素的个数； 
   - Counter —— 计数器，在一些情况下，会比Gauge高效，比如通过一个AtomicLong变量来统计一个队列的长度； 
   - Meter —— 吞吐量的度量，也就是一系列事件发生的速率，例如TPS； 
   - Histogram —— 度量值的统计结果，如最大值、最小值、平均值，以及分布情况等。
