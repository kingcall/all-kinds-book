[toc]

# 任务提交与

## flink框架结构
![image](04E95249172D45C69A12B6A7E3B49FEE)
> 当 Flink 集群启动后，首先会启动一个JobManger和一个或多个的 TaskManager。由 Client 提交任务给 JobManager，JobManager 再调度任务到各个TaskManager去执行，然后TaskManager将心跳和统计信息汇报给JobManager。TaskManager之间以流的形式进行数据的传输。上述三者均为独立的 JVM 进程。


### master
- Master 部分又包含了三个组件，即 Dispatcher、ResourceManager
和 JobManager。其中，Dispatcher 负责接收用户提供的作业，并且负责为这个
新提交的作业拉起一个新的 JobManager 组件。ResourceManager 负责资源的
管理，在整个 Flink 集群中只有一个 ResourceManager。JobManager 负责管理
作业的执行，在一个 Flink 集群中可能有多个作业同时执行，每个作业都有自己的
JobManager 组件。这三个组件都包含在 AppMaster 进程中。

#### JobManager
- JobManager扮演着集群中的管理者Master的角色，它是整个集群的协调者，负责接收Flink Job，协调检查点，Failover 故障恢复等，同时管理Flink集群中从节点TaskManager。
- 主进程接受一个可执行的应用以及一个打包了全部资源文件的jar
- JobManager 从 ResourcesManager申请和接收资源(TaskManager的slot),RM把TM空闲的slots进行分配，JM一旦分配到足够的资源，就把任务分发到对应TM上去执行。在任务执行过程中，jm也会响应所有的请求动作：比如savepoint、checkpoint等。
- JobManager将JobGraph 转化成 ExecutionGraph，然后将任务分配到TaskManager 上执行

#### ResourcesManager
- ResourceManager管理TaskManager的solts(flink资源单位)，根据不同的执行环境，提供了不同的资源管理实现，比如Yarn、Standalone都有各自的实现方式
- Resource Manager 负责整个 Flink Cluster 的资源调度以及与外部调度系统对接，这里的外部调度系统指的是 Kubernetes、Mesos、Yarn 等资源管理系统。
- RM 会首先向 TaskManager 发送 RPC
要求将选定的 Slot 分配给特定的 JobManager。
- TaskManager 如果还没有执行过
该 JobManager 的 Task 的话，它需要首先向相应的 JobManager 建立连接，然
后发送提供 Slot 的 RPC 请求
- 在 JobManager 中，所有 Task 的请求会缓存到
SlotPool 中。当有 Slot 被提供之后，SlotPool 会从缓存的请求中选择相应的请求并
结束相应的请求过程。
- 当 Task 结束之后，无论是正常结束还是异常结束，都会通知 JobManager 相
应的结束状态，然后在 TaskManager 端将 Slot 标记为已占用但未执行任务的状
态。JobManager 会首先将相应的 Slot 缓存到 SlotPool 中，但不会立即释放。这
种方式避免了如果将 Slot 直接还给 ResourceManager，在任务异常结束之后需要重启时，需要立刻重新申请 Slot 的问题。通过延时释放，Failover 的 Task 可以
尽快调度回原来的 TaskManager，从而加快 Failover 的速度。
- 当 SlotPool 中缓
存的 Slot 超过指定的时间仍未使用时，SlotPool 就会发起释放该 Slot 的过程。与
申请 Slot 的过程对应，SlotPool 会首先通知 TaskManager 来释放该 Slot，然后
TaskExecutor 通知 ResourceManager 该 Slot 已经被释放，从而最终完成释放的
逻辑。


##### SlotManager
- 在 ResourceManager 中，有一个子组件叫做 SlotManager，它维护了当前集群中所有 TaskExecutor 上的 Slot 的信息与状态，如该 Slot 在哪个TaskExecutor 中，该 Slot 当前是否空闲等。
- TaskExecutor 启动之后，它会通过
服务发现找到当前活跃的 ResourceManager 并进行注册。在注册信息中，会包含
该 TaskExecutor 中所有 Slot 的信息。 ResourceManager 收到注册信息后，其中
的 SlotManager 就会记录下相应的 Slot 信息。
- 当 JobManager 为某个 Task 来申
请资源时， SlotManager 就会从当前空闲的 Slot 中按一定规则选择一个空闲的 Slot
进行分配。当分配完成后

#### Dispatcher
- Dispatcher 会跨应用运行，它提供了一个Rest 接口让我们提交应用，一旦应用提交执行，Dispatcher 就会启动一个JobManager并将应用转交给刚才启动的JobManager
- Dispatcher 还会启动一个web-ui，用来查看作业相关的信息

#### Scheduler
- Job Manager 中的 Scheduler 组件负责调度执行该 Job 的 DAG 中所有 Task ，发出资源请求，即整个资源调度的起点；


### TaskManager
- TaskManager是实际负责执行计算的Worker，在其上执行FlinkJob的一组Task，每个TaskManager负责**管理其所在节点上的资源信息，如内存、磁盘、网络，在启动的时候将资源的状态向JobManager汇报**
- Flink 的工作进程，每个TaskManager拥有一定数量的slot，slot的数目限定了TaskManager可执行的任务数
- TaskManager 启动之后会向ResourcesManager 注册slot
- 当接收到ResourcesManager的指示后就会向JobManager 提供slot
- JobManager 就可以向slot 中分配任务,从 JobManager 处接收需要部署的 Task，部署启动后，与自己的上游建立 Netty 连接，接收数据并处理
- Flink的TaskManager类似于Spark的worker节点，不同的是一个Sparkworker节点上还可以同时跑多个Executor进程（JVM进程），一个Executor进程只会被一个SparkAPP独享，然后Executor里以多线程方式执行具体任务**。Spark这种设计的好处是将各个应用之间的资源消耗进行了隔离，每个应用都运行在它们各自的JVM中**




### client
- Client是Flink程序提交的客户端，**当用户提交一个Flink程序时，会首先创建一个Client**，该Client首先会对用户提交的Flink程序进行预处理，并提交到Flink集群中处理，所以Client需要从用户提交的Flink程序配置中获取JobManager的地址，并建立到JobManager的连接，将FlinkJob提交给JobManager。
- Client会将用户提交的Flink程序组装一个JobGraph，并且是以JobGraph的形式提交的。一个JobGraph是一个FlinkDataflow，它由多个JobVertex组成的DAG。其中，一个JobGraph包含了一个Flink程序的如下信息：JobID、Job名称、配置信息、一组JobVertex等。
- Client 为提交 Job 的客户端，可以是运行在任何机器上（与 JobManager 环境连通即可）。**提交Job后，Client可以结束进程（Streaming的任务），也可以不结束并等待结果返回**。
> 当用户提交作业的时候，提交脚本会首先启动一个 Client 进程
负责作业的编译与提交。它首先将用户编写的代码编译为一个 JobGraph，在这个过程，**它还会进行一些检查或优化等工作**，例如判断哪些 Operator 可以 Chain 到同一个 Task 中。然后，Client 将产生的 JobGraph 提交到集群中执行。

> 此时有两种情况，一种是类似于 Standalone 这种 Session 模式，AM 会预先启动，此时 Client直接与Dispatcher建立连接并提交作业即可。另一种是 Per-Job 模式，AM 不会预先启动，此时 Client 将**首先向资源管理系统（ 如 Yarn、K8S）申请资源来启动AM，然后再向 AM 中的 Dispatcher 提交作业**。

- 当作业到 Dispatcher 后，Dispatcher 会首先启动一个 JobManager 组件，然后 JobManager 会向 ResourceManager 申请资源来启动作业中具体的任务。
- ResourceManager 选择到
空闲的 Slot 之后，就会通知相应的 TM “将该 Slot 分配分 JobManager XX ”，然
后 TaskExecutor 进行相应的记录后，会向 JobManager 进行注册。JobManager
收到 TaskExecutor 注册上来的 Slot 后，就可以实际提交 Task 了。
TaskExecutor

# API
- Apache Flink 在诞生之初的设计哲学是：用同一个引擎支持多种形式的计算，包括批处理、流处理和机器学习等。尤其是在流式计算方面，**Flink 实现了计算引擎级别的流批一体。那么对于普通开发者而言，如果使用原生的 Flink ，直接的感受还是要编写两套代码**。

![image](98A51D572AF94A008AFCB6456E090C46)
- 对于 DataSet 而言，Source 部分来源于文件、表或者 Java 集合；而 DataStream 的 Source 部分则一般是消息中间件比如 Kafka 等。


## Flink job启动方式
- 提交job(submitJob) 方式， 由开发者提交应用时候触发
- 恢复job 方式(recoverJob), 由Flink 系统内部触发，在JobManager crash情况下触发

![image-20210202090746014](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202090746014.png)

- 第1步，作业提交是通过 Flink Yarn Client，将用户所写的作业代码以及编译好的 jar 包上传到 HDFS 上；
- 第2步 Flink Client 与 Yarn ResourceManager 进行通信，申请所需要的的 Container 资源；
- 第3步，ResourceManager 收到请求后会在集群中的 NodeManager 分配启动 AppMaster 的 Container 进程，AppMaster 中包含 Flink JobManager 模块和 Yarn 通信的 ResourceManager 模块；
- 第4步，在 JobManager 中根据作业的 JobGraph 生成 Execution Graph，ResourceManager 模块向 Yarn 的 ResourceManager 通信，申请 TaskManager 需要的 container 资源，这些 container 由 Yarn 的 NodeManger 负责拉起。
- 第5步 每个 NodeManager 从 HDFS 上下载资源，启动 Container(TaskManager)，并向 JobManager 注册；JobManger 会部署不同的 task 任务到各个 TaskManager 中执行。

## Application 提交
- Flink 应用程序的执行包括两个阶段：pre-flight，即当用户的 main() 方法被调用时；runtime，即用户代码调用 execute()时立即触发。main() 方法使用 Flink 的 API（DataStream API、Table API、DataSet API）之一构造用户程序。当 main() 方法调用 env.execute() 时，**用户定义的管道将被转换成一种 Flink 运行时可以理解的形式，称为 Job Graph（作业图），并将其传递给集群**。
- 尽管它们方法有所不同，Session 模式和 Per-Job 模式都会在 Client 执行应用程序的 main() 方法，即 pre-flight 阶段。

## Session 模式

Session 模式（会话模式）假定集群已经运行，并使用该集群的资源来执行任何提交的应用程序。在同一（Session）集群中执行的应用程序使用相同的资源，并因此相互竞争。这样做的好处是，你无需为每个提交的作业分配整个集群的资源开销。但是，如果其中一个作业行为不正常或者关闭了 TaskManager，那么在该 TaskManager 上运行的所有作业都将受到故障的影响。除了对导致故障的作业产生负面影响之外，这还意味着潜在的大规模恢复过程，即所有重新启动的作业同时访问文件系统，并使其不可用于其他服务。此外，单个集群运行多个作业意味着 JobManager 的负载更大，它负责集群中所有作业的 bookkeeping。这种模式非常适合启动短作业，例如交互式查询。

## Per-Job 模式
- 在 Per-Job 模式中，可用的集群管理器框架（如 YARN 或 Kubernetes）用于为每个提交的作业启动 Flink 集群，该集群仅对该作业可用
- 当作业完成后，集群将关闭，并清理所有延迟的资源（例如文件）。
- 这种模式允许更好的资源隔离，因为行为不正常的作业不会影响任何其他作业。另外，由于每个应用程序都有自己的 JobManager，因此它将 bookkeeping 负载分散到多个实体。考虑到前面提到的 Session模式中的资源隔离问题，对于长时间运行的作业，用户经常选择 Per-Job 模式，因为这些作业愿意接受一定程度的启动延迟的增加，以支持弹性。
- Per-job 模式下整个
Flink 集群只执行单个作业，即每个作业会独享 Dispatcher 和 ResourceManager
组件
- 此外，Per-job 模式下 AppMaster 和 TaskExecutor 都是按需申请的。因
此，Per-job 模式**更适合运行执行时间较长的大作业**，这些作业对稳定性要求较
高，并且对申请资源的时间不敏感
- 总之在 Session 模式中，**集群生命周期独立于集群中运行的任何作业，并且集群中运行的所有作业共享其资源**。
- Per-Job 模式选择为每个提交的作业分配一个集群，已提供更好的资源隔离保证，因为资源不会在作业之间共享。在这种情况下，集群的生命周期与作业的生命周期相关。
- ，在 Session 模式下，Flink 预先启动
AppMaster 以及一组 TaskExecutor，然后在整个集群的生命周期中会执行多个作业。**可以看出，Session 模式更适合规模小，执行时间短的作业**。