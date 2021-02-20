[toc]

## 背景
- Flink 提供的 Metrics 可以在 Flink 内部收集一些指标，通过这些指标让开发人员更好地理解作业或集群的状态
- Metrics是系统内部暴露出的一些指标数据，反应系统当前的运行状态，常用于管理员或用户来衡量应用的健康状态

## Metric Types

### Counter
- 用与存储数值类型，比如统计数据输入、输出总数量
- 其实含义都是一样的，就是对一个计数器进行累加，即对于多条数据和多兆数据一直往上加的过程。

### Gauge
- 指标值，用来记录一个metrics的**瞬间值**(车的仪表盘上的速度)
- Gauge 是最简单的 Metrics，它反映一个值。比如要看现在 Java heap， 内存用了多少，就可以每次实时的暴露一个 Gauge，Gauge 当前的值就是heap使用的量。
- 可以用来存储任何类型，前提要实现org.apache.flink.metrics.Gauge接口，重写getValue方法，如果返回类型为Object则该类需要重写toString方法。


### Meter 
- 是指统计吞吐量和单位时间内发生“事件”的次数。它相当于求一种速率，即事件次数除以使用的时间。
- 用来计算平均速率，直接使用其子类MeterView更方便一些

### Histogram
- Histogram 比较复杂，也并不常用，Histogram用于统计一些数据的分布，比如说Quantile、Mean、StdDev、Max、Min 等
- Flink中属于Histogram的指标很少，但是最重要的一个是属于operator的latency。此项指标会记录数据处理的延迟信息，对任务监控起到很重要的作用。


## Metric Group
- Metric 在 Flink 内部有多层结构，以 Group 的方式组织，它并不是一个扁平化的结构，Metric Group + Metric Name 是 Metrics 的唯一标识
- Metric Group 的层级有 TaskManagerMetricGroup 和TaskManagerJobMetricGroup
```
•TaskManagerMetricGroup
    •TaskManagerJobMetricGroup
        •TaskMetricGroup
            •TaskIOMetricGroup
            •OperatorMetricGroup
                •${User-defined Group} / ${User-defined Metrics}
                •OperatorIOMetricGroup
•JobManagerMetricGroup
    •JobManagerJobMetricGroup
```
### flink Metrics 代码解析
- link中先会定义好ScopeFormat，scopeFormat定义了各类组件metrics_group的范围，然后各个组件（JobManager，TaskManager，Operator等）都会继承ScopeFormat类进行各自的format实现。

![image-20210202201540996](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201540996.png)
- 而后开始定义各个组件的metricsGroup。每个group中定义属于这个组件中所有的metrics。比如TaskIOMetricGroup类，就定义了task执行过程中有关IO的metrics。
- 定义好各个metricsGroup后，在初始化各个组件的时候，会将相应的metricsGroup当做参数放入构造函数中进行初始化。

## 使用Metrics
### System Metrics
### User-defined Metrics
- Datastream 的 API 是继承 RichFunction ，继承 RichFunction 才可以有 Metrics 的接口。
- 然后通过 RichFunction 会带来一个 getRuntimeContext().getMetricGroup().addGroup(…) 的方法，这里就是 User-defined Metrics 的入口。通过这种方式，可以自定义 user-defined Metric Group。如果想定义具体的 Metrics，同样需要用getRuntimeContext().getMetricGroup().counter/gauge/meter/histogram(…) 方法，它会有相应的构造函数，可以定义到自己的 Metrics 类型中

![image-20210202201600493](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201600493.png)

## 获取 Metric
### WebUI
- 首先可以在 WebUI 上看到

![image-20210202201620794](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201620794.png)

###  RESTful API
- 其次可以通过 RESTfulAPI获取，RESTfulAPI对程序比较友好，比如写自动化脚本或程序，自动化运维和测试，通过 RESTful API解析返回的 Json 格式对程序比较友好；
- /jobmanager/metrics 返回jobmanager相关metrics列表 
- /taskmanagers/<taskmanagerid>/metrics
- /jobs/<jobid>/metrics
- /jobs/<jobid>/vertices/<vertexid>/subtasks/<subtas

###  Metric Reporter
- 最后，还可以通过 Metric Reporter 获取，监控主要使用 Metric Reporter 功能。

#### 配置 Metric Reporter 
- metrics.reporters: your_monitor,jmx
- metrics.reporter.jmx.class: org.apache.flink.metrics.jmx.JMXReporter
- metrics.reporter.jmx.port: 1025-10000
- metrics.reporter.your_monitor.class: com.your_company.YourMonitorClass
- metrics.reporter.your_monitor.interval: 10 SECONDS
- metrics.reporter.your_monitor.config.a: your_a_value
- metrics.reporter.your_monitor.config.b: your_b_value- 

#### 常见的 Metric Reporter
- Flink 内置了很多 Reporter，对外部系统的技术选型可以参考，比如 JMX 是 java 自带的技术，不严格属于第三方。还有InfluxDB、Prometheus、Slf4j（直接打 log 里）等，调试时候很好用，可以直接看 logger，Flink 本身自带日志系统，会打到 Flink 框架包里面去
- 我们可以自定义实现metric库，来导入到自己的系统.

##### kafka Metric Reporter
![image-20210202201829101](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201829101.png)

##### 自定义Reporter
##### Jmx
##### Ganglia
##### Graphite
##### influxdb

## 实战
### 作业的延迟监控
1. Source定时向下游发送latencyMarker
2. Latencymarker向下游流转的过程中不会超越records
3. Latencymarker不会记录operator算子的计算时长
4. 每个算子维护一份latency列表用于衡量延迟
5. 整体延迟在500ms以下

![image-20210202201814530](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201814530.png)


## 扩展

### 社区重要指标含义解释



系统类指标

![image-20210202201751867](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201751867.png)



checkpoint 相关

![image-20210202201723041](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201723041.png)



流量相关

![image-20210202201654764](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202201654764.png)