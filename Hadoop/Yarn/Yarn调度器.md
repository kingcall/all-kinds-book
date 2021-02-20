[toc]

![Yarn调度器](http://qiniu.ikeguang.com/image/2021/01/27/23:16:28-Yarn%E8%B0%83%E5%BA%A6%E5%99%A8.jpg)

理想情况下，我们应用对Yarn资源的请求应该立刻得到满足，但现实情况资源往往是有限的，特别是在一个很繁忙的集群，一个应用资源的请求经常需要等待一段时间才能的到相应的资源。在Yarn中，负责给应用分配资源的就是Scheduler。其实调度本身就是一个难题，很难找到一个完美的策略可以解决所有的应用场景。为此，Yarn提供了多种调度器和可配置的策略供我们选择。YARN架构如下:

![1](http://qiniu.ikeguang.com/image/2021/01/27/22:42:14-1.png)

- ResourceManager（RM）：负责对各NM上的资源进行统一管理和调度，将AM分配空闲的Container运行并监控其运行状态。对AM申请的资源请求分配相应的空闲Container。主要由两个组件构成：调度器（Scheduler）和应用程序管理器（Applications Manager）。
- 调度器（Scheduler）：调度器根据容量、队列等限制条件（如每个队列分配一定的资源，最多执行一定数量的作业等），将系统中的资源分配给各个正在运行的应用程序。调度器仅根据各个应用程序的资源需求进行资源分配，而资源分配单位是Container，从而限定每个任务使用的资源量。Scheduler不负责监控或者跟踪应用程序的状态，也不负责任务因为各种原因而需要的重启（由ApplicationMaster负责）。总之，调度器根据应用程序的资源要求，以及集群机器的资源情况，为用程序分配封装在Container中的资源。调度器是可插拔的，例如CapacityScheduler、FairScheduler。（PS：在实际应用中，只需要简单配置即可）
- 应用程序管理器（Application Manager）：应用程序管理器负责管理整个系统中所有应用程序，包括应用程序提交、与调度器协商资源以启动AM、监控AM运行状态并在失败时重新启动等，跟踪分给的Container的进度、状态也是其职责。ApplicationMaster是应用框架，它负责向ResourceManager协调资源，并且与NodeManager协同工作完成Task的执行和监控。MapReduce就是原生支持的一种框架，可以在YARN上运行Mapreduce作业。有很多分布式应用都开发了对应的应用程序框架，用于在YARN上运行任务，例如Spark，Storm等。如果需要，我们也可以自己写一个符合规范的YARN application。
- NodeManager（NM）：NM是每个节点上的资源和任务管理器。它会定时地向RM汇报本节点上的资源使用情况和各个Container的运行状态；同时会接收并处理来自AM的Container 启动/停止等请求。ApplicationMaster（AM）：用户提交的应用程序均包含一个AM，负责应用的监控，跟踪应用执行状态，重启失败任务等。
- Container：是YARN中的资源抽象，它封装了某个节点上的多维度资源，如内存、CPU、磁盘、网络等，当AM向RM申请资源时，RM为AM返回的资源便是用Container 表示的。YARN会为每个任务分配一个Container且该任务只能使用该Container中描述的资源。
# 1. Yarn调度器介绍
## 1.1. FIFO Scheduler(先进先出调度器)

FIFO Scheduler把应用按提交的顺序排成一个队列，这是一个先进先出队列，在进行资源分配的时候，先给队列中最头上的应用进行分配资源，待最头上的应用需求满足后再给下一个分配，以此类推。FIFO Scheduler是最简单也是最容易理解的调度器，也不需要任何配置，但它并不适用于共享集群。大的应用可能会占用所有集群资源，这就导致其它应用被阻塞。在共享集群中，更适合采用Capacity Scheduler或Fair Scheduler，这两个调度器都允许大任务和小任务在提交的同时获得一定的系统资源。下面“Yarn调度器对比图”展示了这几个调度器的区别，从图中可以看出，**在FIFO 调度器中，小任务会被大任务阻塞**。

![2](http://qiniu.ikeguang.com/image/2021/01/27/22:44:05-2.png)

## 1.2.Capacity Scheduler(容量调度器)

 yarn-site.xml中默认配置的资源调度器。而对于Capacity调度器，有一个专门的队列用来运行小任务，但是为小任务专门设置一个队列会预先占用一定的集群资源，这就导致大任务的执行时间会落后于使用FIFO调度器时的时间。用这个资源调度器，就可以配置yarn资源队列，这个后面后介绍用到。

![3](http://qiniu.ikeguang.com/image/2021/01/27/22:44:54-3.png)

## 1.3. Fair Scheduler(公平调度器)

Fair调度器的设计目标是为所有的应用分配公平的资源（对公平的定义可以通过参数来设置）。在上面的“Yarn调度器对比图”展示了一个队列中两个应用的公平调度；当然，公平调度在也可以在多个队列间工作。举个例子，假设有两个用户A和B，他们分别拥有一个队列。当A启动一个job而B没有任务时，A会获得全部集群资源；当B启动一个job后，A的job会继续运行，不过一会儿之后两个任务会各自获得一半的集群资源。如果此时B再启动第二个job并且其它job还在运行，则它将会和B的第一个job共享B这个队列的资源，也就是B的两个job会用于四分之一的集群资源，而A的job仍然用于集群一半的资源，结果就是资源最终在两个用户之间平等的共享。在Fair调度器中，我们不需要预先占用一定的系统资源，Fair调度器会为所有运行的job动态的调整系统资源。当第一个大job提交时，只有这一个job在运行，此时它获得了所有集群资源；当第二个小任务提交后，Fair调度器会分配一半资源给这个小任务，让这两个任务公平的共享集群资源。
a) 公平调度器，就是能够共享整个集群的资源
b) 不用预先占用资源，每一个作业都是共享的
c) 每当提交一个作业的时候，就会占用整个资源。如果再提交一个作业，那么第一个作业就会分给第二个作业一部分资源，第一个作业也就释放一部分资源。再提交其他的作业时，也同理。。。。也就是说每一个作业进来，都有机会获取资源。

 ![4](http://qiniu.ikeguang.com/image/2021/01/27/22:45:31-4.png)

## 1.4. Fair Scheduler与Capacity Scheduler区别

- **资源公平共享**：在每个队列中，Fair Scheduler可选择按照FIFO、Fair或DRF策略为应用程序分配资源。Fair策略即平均分配，默认情况下，每个队列采用该方式分配资源

- **支持资源抢占**：当某个队列中有剩余资源时，调度器会将这些资源共享给其他队列，而当该队列中有新的应用程序提交时，调度器要为它回收资源。为了尽可能降低不必要的计算浪费，调度器采用了先等待再强制回收的策略，即如果等待一段时间后尚有未归还的资源，则会进行资源抢占；从那些超额使用资源的队列中杀死一部分任务，进而释放资源

- **负载均衡**：Fair Scheduler提供了一个基于任务数的负载均衡机制，该机制尽可能将系统中的任务均匀分配到各个节点上。此外，用户也可以根据自己的需求设计负载均衡机制

- **调度策略灵活配置**：Fiar Scheduler允许管理员为每个队列单独设置调度策略（当前支持FIFO、Fair或DRF三种）

- **提高小应用程序响应时间**：由于采用了最大最小公平算法，小作业可以快速获取资源并运行完成

# 2.Yarn调度器配置
yarn资源调度器是在`yarn-site.xml`中配置。
## 2.1. FairScheduler
 Fair Scheduler的配置选项包括两部分:
 一部分在yarn-site.xml中，主要用于配置调度器级别的参数
 一部分在一个自定义配置文件（默认是fair-scheduler.xml）中，主要用于配置各个队列的资源量、权重等信息。

**2.1.1 yarn-site.xml**

`yarn-site.xml`介绍
```xml
<!– scheduler start –>
<property>
    <name>yarn.resourcemanager.scheduler.class</name>
    <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler</value>
    <description>配置Yarn使用的调度器插件类名；Fair Scheduler对应的是：org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler</description>
</property>
<property>
    <name>yarn.scheduler.fair.allocation.file</name>
    <value>/etc/hadoop/conf/fair-scheduler.xml</value>
    <description>配置资源池以及其属性配额的XML文件路径（本地路径）</description>
</property>
<property>
    <name>yarn.scheduler.fair.preemption</name>
    <value>true</value>
    <description>开启资源抢占,default is True</description>
</property>
<property>
    <name>yarn.scheduler.fair.user-as-default-queue</name>
    <value>true</value>
    <description>设置成true，当任务中未指定资源池的时候，将以用户名作为资源池名。这个配置就实现了根据用户名自动分配资源池。default is True</description>
</property>
<property>
    <name>yarn.scheduler.fair.allow-undeclared-pools</name>
    <value>false</value>
    <description>是否允许创建未定义的资源池。如果设置成true，yarn将会自动创建任务中指定的未定义过的资源池。设置成false之后，任务中指定的未定义的资源池将无效，该任务会被分配到default资源池中。,default is True</description>
</property>
<!– scheduler end –>
```

**2.1.2 fair-scheduler.xml**

假设在生产环境Yarn中，总共有四类用户需要使用集群，production、spark、default、streaming。为了使其提交的任务不受影响，我们在Yarn上规划配置了四个资源池，分别为production,spark,default,streaming。并根据实际业务情况，为每个资源池分配了相应的资源及优先级等,default用于开发测试目的.
ResourceManager上fair-scheduler.xml配置如下：

```xml
<?xml version="1.0"?>
<allocations>
    <queue name="root">
        <aclSubmitApps></aclSubmitApps>
        <aclAdministerApps></aclAdministerApps>
        <queue name="production">
            <minResources>8192mb,8vcores</minResources>
            <maxResources>419840mb,125vcores</maxResources>
            <maxRunningApps>60</maxRunningApps>
            <schedulingMode>fair</schedulingMode>
            <weight>7.5</weight>
            <aclSubmitApps>*</aclSubmitApps>
            <aclAdministerApps>production</aclAdministerApps>
        </queue>
        <queue name="spark">
            <minResources>8192mb,8vcores</minResources>
            <maxResources>376480mb,110vcores</maxResources>
            <maxRunningApps>50</maxRunningApps>
            <schedulingMode>fair</schedulingMode>
            <weight>1</weight>
            <aclSubmitApps>*</aclSubmitApps>
            <aclAdministerApps>spark</aclAdministerApps>
        </queue>
        <queue name="default">
            <minResources>8192mb,8vcores</minResources>
            <maxResources>202400mb,20vcores</maxResources>
            <maxRunningApps>20</maxRunningApps>
            <schedulingMode>FIFO</schedulingMode>
            <weight>0.5</weight>
            <aclSubmitApps>*</aclSubmitApps>
            <aclAdministerApps>*</aclAdministerApps>
        </queue>
        <queue name="streaming">
            <minResources>8192mb,8vcores</minResources>
            <maxResources>69120mb,16vcores</maxResources>
            <maxRunningApps>20</maxRunningApps>
            <schedulingMode>fair</schedulingMode>
            <aclSubmitApps>*</aclSubmitApps>
            <weight>1</weight>
            <aclAdministerApps>streaming</aclAdministerApps>
        </queue>
    </queue>
    <user name="production">
        <!-- 对于特定用户的配置:production最多可以同时运行的任务 -->
        <maxRunningApps>100</maxRunningApps>
    </user>
    <user name="default">
        <!-- 对于默认用户配置最多可以同时运行的任务 -->
        <maxRunningApps>10</maxRunningApps>
    </user>

    <!-- users max running apps -->
    <userMaxAppsDefault>50</userMaxAppsDefault>
    <!--默认的用户最多可以同时运行的任务 -->
    <queuePlacementPolicy>
        <rule name="specified"/> 
        <rule name="primaryGroup" create="false" />
        <rule name="secondaryGroupExistingQueue" create="false" />
        <rule name="default" queue="default"/>
    </queuePlacementPolicy>
</allocations>
```

**参数介绍:**
- minResources:最少资源保证量，设置格式为“X mb, Y vcores”，当一个队列的最少资源保证量未满足时，它将优先于其他同级队列获得资源，对于不同的调度策略（后面会详细介绍），最少资源保证量的含义不同，对于fair策略，则只考虑内存资源，即如果一个队列使用的内存资源超过了它的最少资源量，则认为它已得到了满足；对于drf策略，则考虑主资源使用的资源量，即如果一个队列的主资源量超过它的最少资源量，则认为它已得到了满足。
- maxResources:最多可以使用的资源量，fair scheduler会保证每个队列使用的资源量不会超过该队列的最多可使用资源量。
- maxRunningApps:最多同时运行的应用程序数目。通过限制该数目，可防止超量Map Task同时运行时产生的中间输出结果撑爆磁盘。
- weight:资源池权重,主要用在资源共享之时，weight越大，拿到的资源越多。比如一个pool中有20GB内存用不了，这时候可以共享给其他pool，其他每个pool拿多少，就是由权重决定的
- aclSubmitApps:可向队列中提交应用程序的Linux用户或用户组列表，默认情况下为“*”，表示任何用户均可以向该队列提交应用程序。需要注意的是，该属性具有继承性，即子队列的列表会继承父队列的列表。配置该属性时，用户之间或用户组之间用“，”分割，用户和用户组之间用空格分割，比如“user1, user2 group1,group2”。
aclAdministerApps:允许管理任务的用户名和组；一个队列的管理员可管理该队列中的资源和应用程序，比如可杀死任意应用程序。
- minSharePreemptionTimeout ：最小共享量抢占时间。如果一个资源池在该时间内使用的资源量一直低于最小资源量，则开始抢占资源。
- schedulingMode/schedulingPolicy：队列采用的调度模式，可以是fifo、fair或者drf。
管理员也可为单个用户添加maxRunningJobs属性限制其最多同时运行的应用程序数目。此外，管理员也可通过以下参数设置以上属性的默认值：
- userMaxJobsDefault：用户的maxRunningJobs属性的默认值。
- defaultMinSharePreemptionTimeout ：队列的minSharePreemptionTimeout属性的默认值。
- defaultPoolSchedulingMode：队列的schedulingMode属性的默认值。
- fairSharePreemptionTimeout：公平共享量抢占时间。如果一个资源池在该时间内使用资源量一直低于公平共享量的一半，则开始抢占资源。
这样，每个用户组下的用户提交任务时候，会到相应的资源池中，而不影响其他业务。队列的层次是通过嵌套<queue>元素实现的。所有的队列都是root队列的孩子，即使没有配到<root>元素里。Fair调度器中的队列有一个权重属性（这个权重就是对公平的定义），并把这个属性作为公平调度的依据。在这个例子中，当调度器分配集群7.5,1,1,0.5资源给production,spark,streaming,default时便视作公平,这里的权重并不是百分比。注意，对于在没有配置文件时按用户自动创建的队列，它们仍有权重并且权重值为1。每个队列内部仍可以有不同的调度策略。队列的默认调度策略可以通过顶级元素<defaultQueueSchedulingPolicy>进行配置，如果没有配置，默认采用公平调度。尽管是Fair调度器，其仍支持在队列级别进行FIFO调度。每个队列的调度策略可以被其内部的<schedulingPolicy> 元素覆盖，在上面这个例子中，default队列就被指定采用fifo进行调度，所以，对于提交到default队列的任务就可以按照FIFO规则顺序的执行了。需要注意，spark,production,streaming,default之间的调度仍然是公平调度。每个队列可配置最大、最小资源占用数和最大可运行的应用的数量。

Fair调度器采用了一套基于规则的系统来确定应用应该放到哪个队列。在上面的例子中，<queuePlacementPolicy> 元素定义了一个规则列表，其中的每个规则会被逐个尝试直到匹配成功。例如，上例第一个规则specified，则会把应用放到它指定的队列中，若这个应用没有指定队列名或队列名不存在，则说明不匹配这个规则，然后尝试下一个规则。primaryGroup规则会尝试把应用放在以用户所在的Unix组名命名的队列中，如果没有这个队列，不创建队列转而尝试下一个规则。当前面所有规则不满足时，则触发default规则，把应用放在default队列中。
当然，我们可以不配置queuePlacementPolicy规则，调度器则默认采用如下规则：
```xml
<queuePlacementPolicy>
      <rule name="specified" />
      <rule name="user" />
</queuePlacementPolicy>
```
上面规则意思是除非队列被准确的定义，否则会以用户名为队列名创建队列。还有一个简单的配置策略可以使得所有的应用放入同一个队列（default），这样就可以让所有应用之间平等共享集群而不是在用户之间。这个配置的定义如下：
```xml
<queuePlacementPolicy>
     <rule name="default" />
</queuePlacementPolicy>
```
实现上面功能我们还可以不使用配置文件，直接设置yarn.scheduler.fair.user-as-default-queue=false，这样应用便会被放入default 队列，而不是各个用户名队列。另外，我们还可以设置yarn.scheduler.fair.allow-undeclared-pools=false，这样用户就无法创建队列了。

当一个job提交到一个繁忙集群中的空队列时，job并不会马上执行，而是阻塞直到正在运行的job释放系统资源。为了使提交job的执行时间更具预测性（可以设置等待的超时时间），Fair调度器支持抢占。抢占就是允许调度器杀掉占用超过其应占份额资源队列的containers，这些containers资源便可被分配到应该享有这些份额资源的队列中。需要注意抢占会降低集群的执行效率，因为被终止的containers需要被重新执行。可以通过设置一个全局的参数yarn.scheduler.fair.preemption=true来启用抢占功能。此外，还有两个参数用来控制抢占的过期时间（这两个参数默认没有配置，需要至少配置一个来允许抢占Container）：

```
minSharePreemptionTimeout
fairSharePreemptionTimeout
```

如果队列在minimum share preemption timeout指定的时间内未获得最小的资源保障，调度器就会抢占containers。我们可以通过配置文件中的顶级元素<defaultMinSharePreemptionTimeout>为所有队列配置这个超时时间；我们还可以在<queue>元素内配置<minSharePreemptionTimeout>元素来为某个队列指定超时时间。

与之类似，如果队列在fair share preemption timeout指定时间内未获得平等的资源的一半（这个比例可以配置），调度器则会进行抢占containers。这个超时时间可以通过顶级元素<defaultFairSharePreemptionTimeout>和元素级元素<fairSharePreemptionTimeout>分别配置所有队列和某个队列的超时时间。上面提到的比例可以通过<defaultFairSharePreemptionThreshold>(配置所有队列)和<fairSharePreemptionThreshold>(配置某个队列)进行配置，默认是0.5。

需要注意的是，所有客户端提交任务的用户和用户组的对应关系，需要维护在ResourceManager上，ResourceManager在分配资源池时候，是从ResourceManager上读取用户和用户组的对应关系的，否则就会被分配到default资源池。在日志中出现”UserGroupInformation: No groups available for user”类似的警告。而客户端机器上的用户对应的用户组无关紧要。

 每次在ResourceManager上新增用户或者调整资源池配额后，需要执行下面的命令刷新使其生效.
 ```
yarn rmadmin -refreshQueues
yarn rmadmin -refreshUserToGroupsMappings
 ```
动态更新只支持修改资源池配额，如果是新增或减少资源池，则需要重启Yarn集群.

 Fair Scheduer各资源池配置及使用情况，在ResourceManager的WEB监控页面上也可以看到: http://ResourceManagerHost:8088/cluster/scheduler

![5](http://qiniu.ikeguang.com/image/2021/01/27/22:46:21-5.png)

## 2.2 Capacity Scheduler配置（默认配置）
hadoop2.7默认使用的是Capacity Scheduler容量调度器
yarn-site.xml
```xml
<property>
  <name>yarn.resourcemanager.scheduler.class</name>
  <value>org.apache.hadoop.yarn.server.resourcemanager.capacity.CapacityScheduler</value>
</property>
```
Capacity 调度器允许多个组织共享整个集群，每个组织可以获得集群的一部分计算能力。通过为每个组织分配专门的队列，然后再为每个队列分配一定的集群资源，这样整个集群就可以通过设置多个队列的方式给多个组织提供服务了。除此之外，队列内部又可以垂直划分，这样一个组织内部的多个成员就可以共享这个队列资源了，在一个队列内部，资源的调度是采用的是先进先出(FIFO)策略。

一个job可能使用不了整个队列的资源。然而如果这个队列中运行多个job，如果这个队列的资源够用，那么就分配给这些job，如果这个队列的资源不够用了呢？其实Capacity调度器仍可能分配额外的资源给这个队列，这就是“弹性队列”(queue elasticity)的概念。

在正常的操作中，Capacity调度器不会强制释放Container，当一个队列资源不够用时，这个队列只能获得其它队列释放后的Container资源。当然，我们可以为队列设置一个最大资源使用量，以免这个队列过多的占用空闲资源，导致其它队列无法使用这些空闲资源，这就是”弹性队列”需要权衡的地方。

假设我们有如下层次的队列：
```
root
├── prod
└── dev
    ├── eng
    └── science
```
下面是一个简单的Capacity调度器的配置文件，文件名为capacity-scheduler.xml。在这个配置中，在root队列下面定义了两个子队列prod和dev，分别占40%和60%的容量。需要注意，一个队列的配置是通过属性yarn.sheduler.capacity.<queue-path>.<sub-property>指定的，<queue-path>代表的是队列的继承树，如root.prod队列，<sub-property>一般指capacity和maximum-capacity。

```xml
<?xml version="1.0"?>
<configuration>
	<property>
		<name>yarn.scheduler.capacity.root.queues(/&eae)
		<value>prod,dev</value>
	</property>
	<property>
		<name>yarn.scheduler.capacity.root.dev.queues</tta*e> 
		<value>eng,science</value>
	</property>
	<property>
		<name>yarn.scheduler.capacity.root.prod.capacity</name>
		<value>40</value>
	</property>
	<property>
		<name>yarn.scheduler.capacity.root.dev.capacity</name>
		<value >60</value>
	</property>
	<property>
		<name>yarn.scheduler.capacity.root.dev.maximuin-capacity</name>
		<value>75</value>
	</property>
	<property>
		<name>yarn.scheduler.capacity.root.dev.eng.capacity</name>
		<value >50</value>
	</property>
	<property>
		<name>yarn.scheduler.capacity.root.dev.science.capacity</name>
		<value >50</value>
	</property>
</configuration>
```

我们可以看到，dev队列又被分成了eng和science两个相同容量的子队列。dev的maximum-capacity属性被设置成了75%，所以即使prod队列完全空闲dev也不会占用全部集群资源，也就是说，prod队列仍有25%的可用资源用来应急。我们注意到，eng和science两个队列没有设置maximum-capacity属性，也就是说eng或science队列中的job可能会用到整个dev队列的所有资源（最多为集群的75%）。而类似的，prod由于没有设置maximum-capacity属性，它有可能会占用集群全部资源。Capacity容器除了可以配置队列及其容量外，我们还可以配置一个用户或应用可以分配的最大资源数量、可以同时运行多少应用、队列的ACL认证等。

关于队列的设置，这取决于我们具体的应用。比如，在MapReduce中，我们可以通过mapreduce.job.queuename属性指定要用的队列。如果队列不存在，我们在提交任务时就会收到错误。如果我们没有定义任何队列，所有的应用将会放在一个default队列中。

注意：对于Capacity调度器，我们的队列名必须是队列树中的最后一部分，如果我们使用队列树则不会被识别。比如，在上面配置中，我们使用prod和eng作为队列名是可以的，但是如果我们用root.dev.eng或者dev.eng是无效的。

## 2.3 FIFO Scheduler
 `yarn-site.xml`文件
 ```xml
<property>
  <name>yarn.resourcemanager.scheduler.class</name>
  <value>org.apache.hadoop.yarn.server.resourcemanager.fifo.FifoScheduler</value>
</property>
 ```