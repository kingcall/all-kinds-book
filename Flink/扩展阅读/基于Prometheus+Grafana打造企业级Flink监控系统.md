在进入本文之前，我先问大家一个问题，你们公司或者业务系统上是如何对生产集群上的数据同步任务、实时计算任务或者是调度任务本身的执行情况和日志进行监控的呢？可能你会回答是自研或者ELK系统或者Zabbix系统。

今天我们要介绍的主角可能会吊打上面的监控系统哦。

随着容器技术的发展，Kubernetes 已然成为大家追捧的容器集群管理系统。Prometheus 作为生态圈 Cloud Native Computing Foundation（简称：CNCF）中的重要一员，其活跃度仅次于 Kubernetes，现已广泛用于 Kubernetes 集群的监控系统中。

在 Flink 任务的监控上，本文将简要介绍 Prometheus 体系中的组件如何使用，实例演示 Prometheus 的安装，配置及使用。并最终形成一套 Flink 任务监控的解决方案。

#### Prometheus来龙去脉

Prometheus 是由前 Google 工程师从 2012 年开始在 Soundcloud 以开源软件的形式进行研发的系统监控和告警工具包，自此以后，许多公司和组织都采用了 Prometheus 作为监控告警工具。Prometheus 的开发者和用户社区非常活跃，它现在是一个独立的开源项目，可以独立于任何公司进行维护。为了证明这一点，Prometheus 于 2016 年 5 月加入 CNCF 基金会，成为继 Kubernetes 之后的第二个 CNCF 托管项目。

最初，Prometheus 被用在微服务系统的监控上，微服务的落地一直都是业界重点关注的问题，其中除了部署难外，最大的问题就是集群监控、系统配置和系统治理等方面的带来的挑战。

2019 年 Flink 横空出世后，随之而来的运维、监控成为大家关注的重点。作为新一代的监控框架，就像网易在实践过程提出的一样，Prometheus 具有以下特点：

- 灵活的数据模型：在Prometheus里，监控数据是由值、时间戳和标签表组成的，其中监控数据的源信息是完全记录在标签表里的；同时Prometheus支持在监控数据采集阶段对监控数据的标签表进行修改，这使其具备强大的扩展能力；
- 强大的查询能力：Prometheus提供有数据查询语言PromQL。从表现上来看，PromQL提供了大量的数据计算函数，大部分情况下用户都可以直接通过PromQL从Prometheus里查询到需要的聚合数据；
- 健全的生态: Prometheus能够直接对常见操作系统、中间件、数据库、硬件及编程语言进行监控；同时社区提供有Java/Golang/Ruby语言客户端SDK，用户能够快速实现自定义监控项及监控逻辑；
- 良好的性能：在性能方面来看，Prometheus提供了PromBench基准测试，从最新测试结果来看，在硬件资源满足的情况下，Prometheus单实例在每秒采集10w条监控数据的情况下，在数据处理和查询方面依然有着不错的性能表现；
- 更契合的架构：采用推模型的监控系统，客户端需要负责在服务端上进行注册及监控数据推送；而在Prometheus采用的拉模型架构里，具体的数据拉取行为是完全由服务端来决定的。服务端是可以基于某种服务发现机制来自动发现监控对象，多个服务端之间能够通过集群机制来实现数据分片。推模型想要实现相同的功能，通常需要客户端进行配合，这在微服务架构里是比较困难的；
- 成熟的社区：Prometheus是CNCF组织第二个毕业的开源项目，拥有活跃的社区；成立至今，社区已经发布了一百多个版本，项目在 GitHub 上获得的star数超过了3.8万。

![图片](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/640-20210219215713657.png)

可以这么说，Prometheus 天生为监控而生。

#### Prometheus架构和组件

Prometheus 的整体架构以及生态系统组件如下图所示：

![图片](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/640-20210219215713856.png)

Prometheus Server 直接从监控目标中或者间接通过推送网关来拉取监控指标，它在本地存储所有抓取到的样本数据，并对此数据执行一系列规则，以汇总和记录现有数据的新时间序列或生成告警。可以通过 Grafana 或者其他工具来实现监控数据的可视化。

Prometheus 生态圈中包含了多个组件，Prometheus 的主要模块包括：Prometheus server, exporters, Pushgateway, PromQL, Alertmanager 以及图形界面。其中许多组件是可选的：

- Prometheus Server: 用于收集和存储时间序列数据。
- Client Library: 客户端库，为需要监控的服务生成相应的 metrics 并暴露给 Prometheus server。当 Prometheus server 来 pull 时，直接返回实时状态的 metrics。
- Push Gateway: 主要用于短期的 jobs。由于这类 jobs 存在时间较短，可能在 Prometheus 来 pull 之前就消失了。为此，这次 jobs 可以直接向 Prometheus server 端推送它们的 metrics。这种方式主要用于服务层面的 metrics，对于机器层面的 metrices，需要使用 node exporter。
- Exporters: 用于暴露已有的第三方服务的 metrics 给 Prometheus。
- Alertmanager: 从 Prometheus server 端接收到 alerts 后，会进行去除重复数据，分组，并路由到对收的接受方式，发出报警。常见的接收方式有：电子邮件，pagerduty，OpsGenie, webhook 等。
- 一些其他的工具。

Prometheus 的工作流程如下：

- Prometheus通过配置文件中指定的服务发现方式来确定要拉取监控指标的目标(Target)。
- 接着从要拉取的目标(应用容器和Pushgateway)，发起HTTP请求到特定的端点(Metric Path)，将指标持久化至本身的TSDB中，TSDB最终会把内存中的时间序列压缩落到硬盘。
- Prometheus会定期通过PromQL计算设置好的告警规则，决定是否生成告警到Alertmanager，后者接收到告警后会负责把通知发送到邮件或企业内部群聊中。

#### Prometheus的数据模型和核心概念

Prometheus 所有采集的监控数据均以指标（metric）的形式保存在内置的时间序列数据库当中（TSDB）：属于同一指标名称，同一标签集合的、有时间戳标记的数据流。除了存储的时间序列，Prometheus 还可以根据查询请求产生临时的、衍生的时间序列作为返回结果。

上面这段话是不是听起来十分拗口？我们用人话来解释一下：

Prometheus 所采集到的数据被定义为【指标】。存储的数据为【时间序列】，所谓时间序列（或称动态数列）是指将同一统计指标的数值按其发生的时间先后顺序排列而成的数列。而存储的数据库为自带的时序数据库TSDB。

##### 指标名称和标签

Prometheus 中每一条时间序列由指标名称（Metrics Name）以及一组标签（键值对）唯一标识。其中指标的名称（metric name）可以反映被监控样本的含义（例如，http_requests_total — 表示当前系统接收到的 HTTP 请求总量），指标名称只能由 ASCII 字符、数字、下划线以及冒号组成，同时必须匹配正则表达式 [a-zA-Z_:][a-zA-Z0-9_:]*。

标签的名称只能由 ASCII 字符、数字以及下划线组成并满足正则表达式 [a-zA-Z_][a-zA-Z0-9_]*。其中以 _作为前缀的标签，是系统保留的关键字，只能在系统内部使用。标签的值则可以包含任何 Unicode 编码的字符。

##### 样本

在时间序列中的每一个点称为一个样本（sample），样本由以下三部分组成：

- 指标（metric）：指标名称和描述当前样本特征的 labelsets；
- 时间戳（timestamp）：一个精确到毫秒的时间戳；
- 样本值（value）：一个 folat64 的浮点型数据表示当前样本的值。

##### 指标类型

Prometheus 的客户端库中提供了四种核心的指标类型。

- Counter：代表一种样本数据单调递增的指标，即只增不减，通常用来统计如服务的请求数，错误数等。
- Gauge：代表一种样本数据可以任意变化的指标，即可增可减，通常用来统计如服务的CPU使用值，内存占用值等。
- Histogram 和 Summary：用于表示一段时间内的数据采样和点分位图统计结果，通常用来统计请求耗时或响应大小等。

讲到这里，读者是不是有所顿悟？还记得 Flink 中的指标类型吗？Flink 也提供了四种类型的监控指标，分别是：Counter、Gauge、Histogram、Meter。

#### Prometheus的安装

我们可以在官网下载Prometheus的安装包：https://prometheus.io/download/ 。这里我们同时安装Prometheus和Grafana，然后进行解压：

```
tar xvfz prometheus-*.tar.gz
cd prometheus-*
```

启动：

```
$ cd prometheus/
// 查看版本
$ ./prometheus --version
// 运行server
$ ./prometheus --config.file=prometheus.yml
```

访问本地的http://localhost:9090/ 即可以看到Prometheus的graph页面。

![图片](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/640-20210219215713958.png)

安装grafana

```
rpm -ivh grafana-6.5.2-1.x86_64.rpm
```

启动：

```
service grafana-server start
```

访问http://localhost:3000/ 可以看到grafana 界面。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2OayUkAYEZMQMOH3nV1Ot1wqEkNleQ8lFEgYm6YzKlbfjpWfSS2G3rTSergvVVHr4X8cGn4xKd5Ag/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当然，Prometheus还有很多其他组件服务于不同的场景，例如pushgateway和nodeexporter。他们各自的作用可以在官网查看。我们暂时不做介绍。

这里假设我们要监控每一个服务器的状态，这时候我们就需要node_manager这个组件。

我们也是直接安装启动：

```
$ tar xvfz node_exporter-xxx.tar.gz
// 进入解压出的目录
$ cd node_exporter-xxx
// 运行监控采集服务
$ ./node_exporter
```

将node_exporter添加到Prometheus服务器，我们请求一下本地的http://localhost:9090/ 可以看到当前机器的一些指标：

![图片](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/640-20210219215714285.png)

总之，如果你要监控不同的目标，那么就需要安装Prometheus体系中不同的组件。关于详细的安装过程和配置过程我们不做过多展开，大家可以网上搜索有非常多的教程。

#### Prometheus+Grafana+nodeManager+pushgateway打造企业级Flink平台监控系统

我们先来看一下整体的监控架构：

![图片](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/640-20210219215714402.png)

这里面有几个核心的组件：

- Flink App ：这是我们需要监控的数据来源
- Pushgateway+nodeManger : 都是Prometheus 生态中的组件，pushGateway服务收集Flink的指标，nodeMnager负责监控运行机器的状态
- Prometheus : 我们监控系统的主角
- Grafana：可视化展示

关于这四个组建的安装，我们不在仔细描述，大家可以参考网上的资源，我们重点讲述一下配置文件。

首先，flink.yaml文件的配置：

```
metrics.reporter.promgateway.class: org.apache.flink.metrics.prometheus.PrometheusPushGatewayReporter
metrics.reporter.promgateway.host: node1
metrics.reporter.promgateway.port: 9091
metrics.reporter.promgateway.jobName: flinkjobs
metrics.reporter.promgateway.randomJobNameSuffix: false
metrics.reporter.promgateway.deleteOnShutdown: true
```

prometheus.yml中的配置：

```
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
        labels:
          instance: 'prometheus'
  - job_name: 'linux'
    static_configs:
      - targets: ['localhost:9100']
        labels:
          instance: 'localhost'
  - job_name: 'pushgateway'
    static_configs:
      - targets: ['localhost:9091']
        labels:
          instance: 'pushgateway'
```

然后我们把 Flink集群、nodeManager、pushGateway、Prometheus、Grafana分别启动起来。

由于上面一句配置好Flink、 nodeManager、pushGateway，并且在Grafana中已经添加了prometheus 数据源，所以Grafana中会自动获取到 flink job的metrics 。我们进入 Grafana 首页，点击New dashboard，创建一个新的dashboard。

![图片](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/640-20210219215714539.png)

选中之后，即会出现对应的监控指标

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/UdK9ByfMT2OayUkAYEZMQMOH3nV1Ot1wOJvsaEDk3sQ0e2dIH9ZA2SVT91EJOCkgq3kPpjyibNRZvjt0zia2kt4g/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

对于 Flink 任务，我们需要监控的指标包括JobManager 服务器状态、Checkpoint情况、程序运行时长、Taskmanager内存，流量。甚至可以加上operator的进出流量用来定位反压问题。

![图片](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/640-20210219215714723.png)

#### 业界典型应用

事实上Prometheus自从一出世，便受到了关注，我们用同程艺龙数据库监控系统的实践来看一下生产上是如何使用Prometheus的。

目前同程的整体监控架构设计如下图所示：

![图片](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/640-20210219215714886.png)

其中几个关键的组件如下：

**Agent**

这是同程用 golang 开发的监控信息采集 agent，负责采集监控指标和实例日志。监控指标包括了该宿主机的相关信息(实例、容器)。

**Pushgateway**

官方提供的组件，因为 Prometheus 是通过 pull 的方式获取数据的，如果让 Prometheus Server 去每个节点拉数据，那么监控服务的压力就会很大，我们是在监控几千个实例的情况下做到 10s 的采集间隔(当然采用联邦集群的模式也可以，但是这样就要需要部署 Prometheus Server。再加上告警相关的东西以后，整个架构会变的比较复杂。)。所以 agent 采取数据推送至 pushgateway，然后由 Prometheus Server 去 pushgateway 上面 pull 数据。这样在 Prometheus Server 在写入性能满足的情况下，单台机器就可以承载整个系统的监控数据。考虑到跨机房采集监控数据的问题，可以在每个机房都部署 pushgateway 节点，同时还能缓解单个 pushgateway 的压力。

**Prometheus Server**

Prometheus Server 去 pushgateway 上面拉数据的时间间隔设置为 10s。多个 pushgateway 的情况下，就配置多个组即可。为了确保 Prometheus Server 的高可用，可以再加一个 Prometheus Server 放到异地容灾机房，配置和前面的 Prometheus Server 一样。如果监控需要保留时间长的话，也可以配置一个采集间隔时间较大的 Prometheus Server，比如 5 分钟一次，数据保留 1 年。

**Alertmanager**

使用 Alertmanager 前，需要先在 Prometheus Server 上面定义好告警规则。支持邮件、微信、webhook 多种类型，告警是通过 webhook 的方式，将触发的告警推送至指定 API，然后通过这个接口的服务进行二次加工。

**Grafana**

Prometheus 完美支持 Grafana，可以通过 PromQL 语法结合 Grafana，快速实现监控图的展示。为了和运维平台关联，通过 url 传参的方式，实现了运维平台直接打开指定集群和指定实例的监控图。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2OayUkAYEZMQMOH3nV1Ot1wJichk95SyEUvsv4vbzcCLsjt23FY55QS2iatzibl8H09GtOp9usnyndKA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)![图片](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/640-20210219215715113.png)

目前同程基于 Prometheus 的监控系统，承载了整个平台所有实例、宿主机、容器的监控。采集周期 10S，Prometheus 一分钟内每秒平均摄取样本数 9-10W。仅仅使用一台物理机(不包括高可用容灾资源)就可以承载当前的流量，并且还有很大的容量空间(CPU\Memory\Disk)。如果未来单机无法支撑的情况下，可以扩容成联邦集群模式。