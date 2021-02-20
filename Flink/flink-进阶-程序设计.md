[toc]
## 资源


## Flink 构建的流程
- 构建计算环境(决定采用哪种计算执行方式)
- 创建Source(可以多个数据源)
- 对数据进行不同方式的转换(提供了丰富的算子) •
- 对结果的数据进行Sink(可以输出到多个地方)

### 算子
1. **Source** : 数据源，Flink 在流处理和批处理上的 source 大概有 4 类：基于本地集合的 source、基于文件的 source、基于网络套接字的 source、自定义的 source。自定义的 source 常见的有 Apache kafka、Amazon Kinesis Streams、RabbitMQ、Twitter Streaming API、Apache NiFi 等，当然你也可以定义自己的 source。
2. **Transformation** : 数据转换的各种操作，有 Map / FlatMap / Filter / KeyBy / Reduce / Fold / Aggregations / Window / WindowAll / Union / Window join / Split / Select / Project 等，操作很多，可以将数据转换计算成你想要的数据。
3. **Sink** : 接收器，Sink 是指 Flink 将转换计算后的数据发送的地点 ，你可能需要存储下来。Flink 常见的 Sink 大概有如下几类：写入文件、打印出来、写入 Socket 、自定义的 Sink 。自定义的 sink 常见的有 Apache kafka、RabbitMQ、MySQL、ElasticSearch、Apache Cassandra、Hadoop FileSystem 等，同理你也可以定义自己的 Sink。

## 批流统一的设计
- 对于开发人员来说，流批统一的引擎（Table API & SQL）在执行之前会根据运行的环境翻译成 DataSet 或者 DataStream
API。因为这两种 API 底层的实现有很大的区别，所以在统一流和批的过程中遇到了不少挑战。

- 理论基础：动态表
- 架构改进（统一的 Operator 框架、统一的查询处理）
- 优化器的统一
- 基础数据结构的统一
- 物理实现的共享
