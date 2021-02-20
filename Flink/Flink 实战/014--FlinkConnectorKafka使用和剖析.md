## FlinkConnectorKafka使用和剖析

[TOC]

在前面 3.6 节中介绍了 Flink 中的 Data Source 和 Data Sink，然后还讲诉了自带的一些 Source 和 Sink 的
Connector。本篇文章将讲解一下用的最多的 Connector —— Kafka，带大家利用 Kafka Connector 读取 Kafka数据，做一些计算操作后然后又通过 Kafka Connector 写入到 kafka 消息队列去。

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-23-101054.png)

### 准备环境和依赖

#### 环境安装和启动

如果你已经安装好了 Flink 和 Kafka，那么接下来使用命令运行启动 Flink、Zookepeer、Kafka 就行了。

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-11-042714.png)

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-11-142523.png)

执行命令都启动好了后就可以添加依赖了。

#### 添加 maven 依赖

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-11-043040.png)

Flink 里面支持 Kafka 0.8.x 以上的版本，具体采用哪个版本的 Maven 依赖需要根据安装的 Kafka 版本来确定。因为之前我们安装的Kafka 是 1.1.0 版本，所以这里我们选择的 Kafka Connector 为 `flink-connector-kafka-0.11_2.11`（支持 Kafka 0.11.x 版本及以上，该 Connector 支持 Kafka 事务消息传递，所以能保证 Exactly Once）。

    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-connector-kafka-0.11_2.11</artifactId>
        <version>${flink.version}</version>
    </dependency>

Flink、Kafka、Flink Kafka Connector 三者对应的版本可以根据[官网](https://ci.apache.org/projects/flink/flink-docs-stable/dev/connectors/kafka.html) 的对比来选择。需要注意的是 `flink-connector-kafka_2.11`这个版本支持的 Kafka 版本要大于 1.0.0，从 Flink 1.9 版本开始，它使用的是 Kafka 2.2.0版本的客户端，虽然这些客户端会做向后兼容，但是建议还是按照官网约定的来规范使用 Connector 版本。另外你还要添加的依赖有：


​    
    <!--flink java-->
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-java</artifactId>
        <version>${flink.version}</version>
        <scope>provided</scope>
    </dependency>
    <dependency>
        <groupId>org.apache.flink</groupId>
        <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
        <version>${flink.version}</version>
        <scope>provided</scope>
    </dependency>
    
    <!--log-->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-log4j12</artifactId>
        <version>1.7.7</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
        <scope>runtime</scope>
    </dependency>
    
    <!--alibaba fastjson-->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.51</version>
    </dependency>


### 测试数据发到 Kafka Topic

实体类，Metric.java


​    
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    public class Metric {
        public String name;
        public long timestamp;
        public Map<String, Object> fields;
        public Map<String, String> tags;
    }


往 kafka 中写数据工具类：KafkaUtils.java


​    
```java
/**
 * 往kafka中写数据，可以使用这个main函数进行测试一下
 */
public class KafkaUtils {
    public static final String broker_list = "localhost:9092";
    public static final String topic = "metric";  // kafka topic，Flink 程序中需要和这个统一 

    public static void writeToKafka() throws InterruptedException {
        Properties props = new Properties();
        props.put("bootstrap.servers", broker_list);
        props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer"); //key 序列化
        props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer"); //value 序列化
        KafkaProducer producer = new KafkaProducer<String, String>(props);

        Metric metric = new Metric();
        metric.setTimestamp(System.currentTimeMillis());
        metric.setName("mem");
        Map<String, String> tags = new HashMap<>();
        Map<String, Object> fields = new HashMap<>();

        tags.put("cluster", "zhisheng");
        tags.put("host_ip", "101.147.022.106");

        fields.put("used_percent", 90d);
        fields.put("max", 27244873d);
        fields.put("used", 17244873d);
        fields.put("init", 27244873d);

        metric.setTags(tags);
        metric.setFields(fields);

        ProducerRecord record = new ProducerRecord<String, String>(topic, null, null, JSON.toJSONString(metric));
        producer.send(record);
        System.out.println("发送数据: " + JSON.toJSONString(metric));

        producer.flush();
    }

    public static void main(String[] args) throws InterruptedException {
        while (true) {
            Thread.sleep(300);
            writeToKafka();
        }
    }
}
```


运行：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-23-101504.png)

如果出现如上图标记的，即代表能够不断往 kafka 发送数据的。

### Flink 如何消费 Kafka 数据？


​    
    public class Main {
        public static void main(String[] args) throws Exception {
            final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
    
            Properties props = new Properties();
            props.put("bootstrap.servers", "localhost:9092");
            props.put("zookeeper.connect", "localhost:2181");
            props.put("group.id", "metric-group");
            props.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");  //key 反序列化
            props.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
            props.put("auto.offset.reset", "latest"); //value 反序列化
    
            DataStreamSource<String> dataStreamSource = env.addSource(new FlinkKafkaConsumer011<>(
                    "metric",  //kafka topic
                    new SimpleStringSchema(),  // String 序列化
                    props)).setParallelism(1);
    
            dataStreamSource.print(); //把从 kafka 读取到的数据打印在控制台
    
            env.execute("Flink add data source");
        }
    }


运行起来：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-23-101832.png)

看到没程序，Flink 程序控制台能够源源不断地打印数据呢。

#### 代码分析

使用 FlinkKafkaConsumer011 时传入了三个参数。

  * Kafka topic：这个代表了 Flink 要消费的是 Kafka 哪个 Topic，如果你要同时消费多个 Topic 的话，那么你可以传入一个 Topic List 进去，另外也支持正则表达式匹配 Topic
  * 序列化：上面代码我们使用的是 SimpleStringSchema
  * 配置属性：将 Kafka 等的一些配置传入 

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-23-102157.png)

前面演示了 Flink 如何消费 Kafak 数据，接下来演示如何把其他 Kafka 集群中 topic 数据原样写入到自己本地起的 Kafka 中去。

### Flink 如何将计算后的数据发到 Kafka？

#### 配置文件


​    
    //其他 Kafka 集群配置
    kafka.brokers=xxx:9092,xxx:9092,xxx:9092
    kafka.group.id=metrics-group-test
    kafka.zookeeper.connect=xxx:2181
    metrics.topic=xxx
    stream.parallelism=5
    kafka.sink.brokers=localhost:9092
    kafka.sink.topic=metric-test
    stream.checkpoint.interval=1000
    stream.checkpoint.enable=false
    stream.sink.parallelism=5


目前我们先看下本地 Kafka 是否有这个 metric-test topic 呢？需要执行下这个命令：


​    
    bin/kafka-topics.sh --list --zookeeper localhost:2181


![](https://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/images/6KFHKT.jpg)

可以看到本地的 Kafka 是没有任何 topic 的，如果等下程序运行起来后，再次执行这个命令出现 metric-test
topic，那么证明程序确实起作用了，已经将其他集群的 Kafka 数据写入到本地 Kafka 了。

#### 程序代码

Main.java

    public class Main {
        public static void main(String[] args) throws Exception{
            final ParameterTool parameterTool = ExecutionEnvUtil.createParameterTool(args);
            StreamExecutionEnvironment env = ExecutionEnvUtil.prepare(parameterTool);
            DataStreamSource<Metrics> data = KafkaConfigUtil.buildSource(env);
    
            data.addSink(new FlinkKafkaProducer011<Metrics>(
                    parameterTool.get("kafka.sink.brokers"),
                    parameterTool.get("kafka.sink.topic"),
                    new MetricSchema()
                    )).name("flink-connectors-kafka")
                    .setParallelism(parameterTool.getInt("stream.sink.parallelism"));
    
            env.execute("flink learning connectors kafka");
        }
    }


#### 运行结果

启动程序，查看运行结果，不段执行上面命令，查看是否有新的 topic 出来：

![](https://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/images/nxqZmZ.jpg)

执行命令可以查看该 topic 的信息：

    bin/kafka-topics.sh --describe --zookeeper localhost:2181 --topic metric-test


![](https://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/images/y5vPRR.jpg)

前面代码使用的 FlinkKafkaProducer011
只传了三个参数：brokerList、topicId、serializationSchema（序列化），其实是支持传入多个参数的。

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-23-102620.png)

### FlinkKafkaConsumer 源码剖析

FlinkKafkaConsumer 的继承关系如下图所示。

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-11-144643.png)

可以发现几个版本的 FlinkKafkaConsumer 都继承自 FlinkKafkaConsumerBase 抽象类，所以可知
FlinkKafkaConsumerBase 是最核心的类了。FlinkKafkaConsumerBase 实现了
CheckpointedFunction、CheckpointListener 接口，继承了 RichParallelSourceFunction
抽象类来读取 Kafka 数据。

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-11-164025.png)

在 FlinkKafkaConsumerBase 中的 open 方法中做了大量的配置初始化工作，然后在 run 方法里面是由
AbstractFetcher 来获取数据的，在 AbstractFetcher 中有用 List> 来存储着所有订阅分区的状态信息，包括了下面这些字段：

    private final KafkaTopicPartition partition;    //分区
    private final KPH kafkaPartitionHandle;
    private volatile long offset;   //消费到的 offset
    private volatile long committedOffset;  //提交的 offset

在 FlinkKafkaConsumerBase 中还有字段定义 Flink 自动发现 Kafka 主题和分区个数的时间，默认是不开启的（时间为Long.MIN_VALUE），**像如果传入的是正则表达式参数，那么动态的发现主题还是有意义的，如果配置的已经是固定的Topic，那么完全就没有开启这个的必要**，另外就是 Kafka 的分区个数的自动发现，像高峰流量的时期，如果 Kafka 的分区扩容了，但是在 Flink这边没有配置这个参数那就会导致 Kafka 新分区中的数据不会被消费到，这个参数由 `flink.partition-discovery.interval-
millis` 控制。

### FlinkKafkaProducer 源码剖析

FlinkKafkaProducer 这个有些特殊，不同版本的类结构有些不一样，如 FlinkKafkaProducer011 是继承的
TwoPhaseCommitSinkFunction 抽象类，而 FlinkKafkaProducer010 和 FlinkKafkaProducer09是基于 FlinkKafkaProducerBase 类来实现的。

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-12-025704.png)

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-12-030003.png)

在 Kafka 0.11.x 版本后支持了事务，这让 Flink 与 Kafka 的事务相结合从而实现端到端的 Exactly once 才有了可能，在9.5 节中会详细讲解如何利用 TwoPhaseCommitSinkFunction 来实现 Exactly once 的。

数据 Sink 到下游的 Kafka，可你能会关心数据的分区策略，在 Flink 中自带了一种就是FlinkFixedPartitioner，它使用的是round-robin 策略进行下发到下游 Kafka Topic 的分区上的，当然也提供了 FlinkKafkaPartitioner接口供你去实现自定义的分区策略。

### 使用 Flink-connector-kafka 可能会遇到的问题

#### 如何消费多个 Kafka Topic

通常可能会有很多类型的数据全部发到 Kafka，但是发送的数据却不是在同一个 Topic 里面，然后在 Flink 处消费的时候，又要去同时消费这些多个Topic，在 Flink 中除了支持可以消费单个 Topic 的数据，还支持传入多个 Topic，另外还支持 Topic的正则表达式（因为有时候可能会事先不确定到底会有多少个 Topic，所以使用正则来处理会比较好，只要在 Kafka 建立的 Topic名是有规律的就行），如下几种构造器可以传入不同参数来创建 FlinkKafkaConsumer 对象。


​    
    //单个 Topic
    public FlinkKafkaConsumer011(String topic, DeserializationSchema<T> valueDeserializer, Properties props) {
        this(Collections.singletonList(topic), valueDeserializer, props);
    }
    
    //多个 Topic
    public FlinkKafkaConsumer011(List<String> topics, DeserializationSchema<T> deserializer, Properties props) {
        this(topics, new KafkaDeserializationSchemaWrapper<>(deserializer), props);
    }
    
    //正则表达式 Topic
    public FlinkKafkaConsumer011(Pattern subscriptionPattern, DeserializationSchema<T> valueDeserializer, Properties props) {
        this(subscriptionPattern, new KafkaDeserializationSchemaWrapper<>(valueDeserializer), props);
    }


#### 想要获取数据的元数据信息

在消费 Kafka 数据的时候，有时候想获取到数据是从哪个 Topic、哪个分区里面过来的，这条数据的 offset
值是多少。这些元数据信息在有的场景真的需要，那么这种场景下该如何获取呢？其实在获取数据进行反序列化的时候使用KafkaDeserializationSchema 就行。


​    
    public interface KafkaDeserializationSchema<T> extends Serializable, ResultTypeQueryable<T> {
    
        boolean isEndOfStream(T nextElement);
    
        T deserialize(ConsumerRecord<byte[], byte[]> record) throws Exception;
    }

在 KafkaDeserializationSchema 接口中的 deserialize 方法里面的 ConsumerRecord类中是包含了数据的元数据信息。

    public class ConsumerRecord<K, V> {
        private final String topic;
        private final int partition;
        private final long offset;
        private final long timestamp;
        private final TimestampType timestampType;
        private final long checksum;
        private final int serializedKeySize;
        private final int serializedValueSize;
        private final K key;
        private final V value;
    }


所在在使用 FlinkKafkaConsumer011 构造对象的的时候可以传入实现 KafkaDeserializationSchema
接口后的参数对象。


​    
    //单个 Topic
    public FlinkKafkaConsumer011(String topic, KafkaDeserializationSchema<T> deserializer, Properties props) {
        this(Collections.singletonList(topic), deserializer, props);
    }
    
    //多个 Topic
    public FlinkKafkaConsumer011(List<String> topics, KafkaDeserializationSchema<T> deserializer, Properties props) {
        super(topics, deserializer, props);
    }
    
    //正则表达式 Topic
    public FlinkKafkaConsumer011(Pattern subscriptionPattern, KafkaDeserializationSchema<T> deserializer, Properties props) {
        super(subscriptionPattern, deserializer, props);
    }


#### 多种数据类型

因为在 Kafka 的数据的类型可能会有很多种类型，比如是纯 String、String 类型的JSON、Avro、Protobuf。那么源数据类型不同，在消费 Kafka 的时候反序列化也是会有一定的不同，但最终还是依赖前面的
KafkaDeserializationSchema 或者 DeserializationSchema （反序列化的Schema），数据经过处理后的结果再次发到 Kafka 数据类型也是会有多种，它依赖的是 SerializationSchema（序列化的Schema）。

#### 序列化失败

因为数据是从 Kafka 过来的，难以避免的是 Kafka 中的数据可能会出现 null或者不符合预期规范的数据，然后在反序列化的时候如果作业里面没有做异常处理的话，就会导致作业失败重启，这样情况可以在反序列化处做异常处理，保证作业的健壮性。

#### Kafka 消费 Offset 的选择

因为在 Flink Kafka Consumer 中是支持配置如何确定从 Kafka 分区开始消费的起始位置的。

    FlinkKafkaConsumer011<String> myConsumer = new FlinkKafkaConsumer0111<>(...);
    consumer.setStartFromEarliest();     //从最早的数据开始消费
    consumer.setStartFromLatest();       //从最新的数据开始消费
    consumer.setStartFromTimestamp(...); //从根据指定的时间戳（ms）处开始消费
    consumer.setStartFromGroupOffsets(); //默认从提交的 offset 开始消费


另外还支持根据分区指定的 offset 去消费 Topic 数据，示例如下：

    Map<KafkaTopicPartition, Long> specificStartOffsets = new HashMap<>();
    specificStartOffsets.put(new KafkaTopicPartition("zhisheng", 0), 23L);
    specificStartOffsets.put(new KafkaTopicPartition("zhisheng", 1), 31L);
    specificStartOffsets.put(new KafkaTopicPartition("zhisheng", 2), 43L);
    myConsumer.setStartFromSpecificOffsets(specificStartOffsets);

注意：这种情况下如果该分区中不存在指定的 Offset 了，则会使用默认的 setStartFromGroupOffsets
来消费分区中的数据。如果作业是从 Checkpoint 或者 Savepoint 还原的，那么上面这些配置无效，作业会根据状态中存储的 Offset为准，然后开始消费。

上面这几种策略是支持可以配置的，需要在作业中指定，具体选择哪种是需要根据作业的业务需求来判断的。

### 小结与反思

本节讲了 Flink 中最常使用的 Connector —— Kafka，该 Connector 不仅可以作为 Source，还可以作为
Sink。通过了完成的案例讲解从 Kafka 读取数据和写入数据到 Kafka，并分析了这两个的主要类的结构。最后讲解了使用该 Connector可能会遇到的一些问题，该如何去解决这些问题。

你在公司使用该 Connector 的过程中有遇到什么问题吗？是怎么解决的呢？还有什么问题要补充？

本节涉及代码地址：<https://github.com/zhisheng17/flink-learning/tree/master/flink-
learning-connectors/flink-learning-connectors-kafka>

