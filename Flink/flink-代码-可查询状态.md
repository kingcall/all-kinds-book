[toc]
## Queryable State
- Queryable State，顾名思义，就是可查询的状态，表示这个状态，在流计算的过程中就可以被查询，而不像其他流计算框架，需要存储到外部系统中才能被查询。目前可查询的state主要针对partitionable state，如keyed state等。
- 当用户在job中定义了queryable state之后，就可以在外部，通过QueryableStateClient，通过job id, state name, key来查询所对应的状态的实时的值。
- 这个设计看起来很美好，通过向流计算实时查询状态数据，免去了传统的存储等的开销。但实际上，除了上面提到的状态类型的限制之外，也会受netty server以及state backend本身的性能限制，因此并不适用于高并发的查询。
- 当Flink应用程序将大量数据推送到外部数据存储时，这可能会成为I / O瓶颈。如果所涉及的数据具有比写入更少的读取，则更好的方法可以是外部应用程序从Flink获取所需的数据。在可查询的状态界面，允许通过Flink被管理的状态，按需要查询支持这个。

### 实现架构
![image](6E93A60031E04D7885206F08AA73F9FA)

```
QueryableStateClient: 客户端。运行在外部系统。提交查询请求并接收最终返回的结果。

QueryableStateClientProxy: 客户端代理。运行在每个TaskManager上。接收客户端的请求，找到Key对应的TaskManager，然后将请求转发给具体的查询服务，并负责最终向客户端返回结果。

QueryableStateServer: 查询服务。运行在每个TaskManager上。处理来自客户端代理的请求并返回结果。
```
### 实现方式一(QueryableStateStream)
- 任务状态数据将会更新到QueryableStateStream *流中，可以理解为是State的一个sink。
- 通过KeyedStream.asQueryableState方法，生成一个QueryableStream，需要注意的是，这个stream类似于一个sink，是不能再做transform的。 实现上，生成QueryableStream就是为当前stream加上一个operator：QueryableAppendingStateOperator，它的processElement方法，每来一个元素，就会调用state.add去更新状态。因此这种方式有一个限制，只能使用ValueDescriptor, FoldingStateDescriptor或者ReducingStateDescriptor，而不能是ListStateDescriptor，因为它可能会无限增长导致OOM。此外，由于不能在stream后面再做transform，也是有一些限制。

### 实现方式二
- 通过managed keyed state
```
ValueStateDescriptor<Tuple2<Long, Long>> descriptor =
new ValueStateDescriptor<>(
      "average", // the state name
      TypeInformation.of(new TypeHint<Tuple2<Long, Long>>() {}),
      Tuple2.of(0L, 0L)); 
  descriptor.setQueryable("query-name");
```
- 这个只需要将具体的state descriptor标识为queryable即可，这意味着可以将一个pipeline中间的operator的state标识为可查询的。
- 首先根据state descriptor的配置，会在具体的TaskManager中创建一个KvStateServer，用于state查询，它就是一个简单的netty server，通过KvStateServerHandler来处理请求，查询state value并返回。
- 但是一个partitionable state，可能存在于多个TaskManager中，因此需要有一个路由机制，当QueryableStateClient给定一个query name和key时，要能够知道具体去哪个TaskManager中查询。
- 为了做到这点，在Job的ExecutionGraph（JobMaster）上会有一个用于定位KvStateServer的KvStateLocationRegistry，当在TaskManager中注册了一个queryable KvStateServer时，就会调用JobMaster.notifyKvStateRegistered，通知JobMaster

### 配置可查询状态

#### yarn
- 添加依赖
```
cp ${FLINK_HOME}/opt/flink-queryable-state-runtime_2.11-1.9.0.jar ${FLINK_HOME}/lib/
```
- 启用Queryable State服务
```
在${FLINK_HOME}/conf/flink-conf.yaml中设置queryable-state.enable: true
```
- 查看TaskManager日志: 在TaskManager中见如下日志，即启动。
```
Started Queryable State Server @ /x.x.x.x:9067.
Started Queryable State Proxy Server @ /x.x.x.x:9069
```

#### local
Flink Local模式可通过如下设置启动。
- 添加依赖
```
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-queryable-state-runtime_2.11</artifactId>
    <version>1.9.0</version>
</dependency>
```

- 启用Queryable State服务
```
Configuration config = new Configuration();
config.setInteger(ConfigOptions.key("rest.port").defaultValue(8081),8081);
config.setBoolean(ConfigConstants.LOCAL_START_WEBSERVER, true);
//启用Queryable State服务
config.setBoolean(QueryableStateOptions.ENABLE_QUERYABLE_STATE_PROXY_SERVER, true);
StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironmentWithWebUI(config);
```
- 查看TaskManager日志: 在TaskManager中见如下日志，即启动。
```
Started Queryable State Server @ /127.0.0.1:9067
Started Queryable State Proxy Server @ /127.0.0.1:9069
```
