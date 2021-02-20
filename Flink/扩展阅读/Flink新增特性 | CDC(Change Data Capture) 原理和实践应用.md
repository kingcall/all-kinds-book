**CDC简介**



CDC,Change Data Capture,变更数据获取的简称，使用CDC我们可以从数据库中获取已提交的更改并将这些更改发送到下游，供下游使用。这些变更可以包括INSERT,DELETE,UPDATE等。

用户可以在以下的场景下使用CDC：

- 使用flink sql进行数据同步,可以将数据从一个数据同步到其他的地方，比如mysql、elasticsearch等。
- 可以在源数据库上实时的物化一个聚合视图
- 因为只是增量同步，所以可以实时的低延迟的同步数据
- 使用EventTime join 一个temporal表以便可以获取准确的结果

Flink 1.11 将这些changelog提取并转化为Table API和SQL，目前支持两种格式：Debezium和Canal，这就意味着源表不仅仅是append操作，而且还有upsert、delete操作。



![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2OtU9SItmD3t5xe2eAcCjAze1kG1HoY8j1rSbRDcMC3QttZ3vCn13LfTxsL0KEBvmCvHp1YaIBmew/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



Flink CDC 功能适用的一些场景：



- 数据库之间的增量数据同步
- 审计日志
- 数据库之上的实时物化视图
- 基于CDC的维表join
- …



# **Flink CDC使用方式**

##  

目前Flink支持两种内置的connector，PostgreSQL和mysql，接下来我们以mysql为例。

![图片](https://mmbiz.qpic.cn/mmbiz_png/UdK9ByfMT2OtU9SItmD3t5xe2eAcCjAzHuqsGjYm8t9hjvsIRiaXQbgGYSHP7GXUAqf4cGlvEM3ogOFQnYxYs2Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)



Flink 1.11仅支持Kafka作为现成的变更日志源和JSON编码的变更日志，而Avro（Debezium）和Protobuf（Canal）计划在将来的版本中使用。还计划支持MySQL二进制日志和Kafka压缩主题作为源，并将扩展日志支持扩展到批处理执行。



**Flink** **CDC当作监听器获取增量变更**



传统的实时链路如何实现业务数据的同步，我们以canal为例，传统业务数据实时同步会涉及到canal处理mysql的binlog然后同步到kafka，在通过计算引擎spark，flink或storm计算转化,再结果数据传输到第三方存储（hbase，es）如下图所示主要分为三个模块E(Extract) ,T(Transform), L(Load).可以看到涉及的组件很多，链路很长。

我们可以直接Flink CDC消费数据库的增量日志，替代了原来作为数据采集层的canal，然后直接进行计算，经过计算之后，将计算结果 发送到下游。整体架构如下：

![image-20210219214854937](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210219214854937.png)

使用这种架构是好处有：

- 减少canal和kafka的维护成本，链路更短，延迟更低
- flink提供了exactly once语义
- 可以从指定position读取
- 去掉了kafka，减少了消息的存储成本

我们需要引入相应的pom，mysql的pom如下：

```
<dependency>
  <groupId>com.alibaba.ververica</groupId>
  <artifactId>flink-connector-mysql-cdc</artifactId>
  <version>1.1.0</version>
</dependency>
```

如果是sql客户端使用，需要下载 flink-sql-connector-mysql-cdc-1.1.0.jar 并且放到<FLINK_HOME>/lib/下面

连接mysql数据库的示例sql如下：

```

-- creates a mysql cdc table source
CREATE TABLE mysql_binlog (
 id INT NOT NULL,
 name STRING,
 description STRING,
 weight DECIMAL(10,3)
) WITH (
 'connector' = 'mysql-cdc',
 'hostname' = 'localhost',
 'port' = '3306',
 'username' = 'flinkuser',
 'password' = 'flinkpw',
 'database-name' = 'inventory',
 'table-name' = 'products'
);
```

使用API的方式：

```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.SourceFunction;
import com.alibaba.ververica.cdc.debezium.StringDebeziumDeserializationSchema;
import com.alibaba.ververica.cdc.connectors.mysql.MySQLSource;
 
public class MySqlBinlogSourceExample {
  public static void main(String[] args) throws Exception {
    SourceFunction<String> sourceFunction = MySQLSource.<String>builder()
      .hostname("localhost")
      .port(3306)
      .databaseList("inventory") // monitor all tables under inventory database
      .username("flinkuser")
      .password("flinkpw")
      .deserializer(new StringDebeziumDeserializationSchema()) // converts SourceRecord to String
      .build();
    StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
    env
      .addSource(sourceFunction)
      .print().setParallelism(1); // use parallelism 1 for sink to keep message ordering
    env.execute();
  }
}
```



**Flink CDC 当作转换工具**

如果需要Flink承担的角色是计算层，那么目前Flink提供的format有两种格式：canal-json和debezium-json，下面我们简单的介绍下。

如果要使用Kafka的canal-json，对于程序而言，需要添加如下依赖:

```
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka_2.11</artifactId>
    <version>1.11.0</version>
</dependency>
```

我们可以直接消费canal-json数据：

```
CREATE TABLE topic_products (
  id BIGINT,
  name STRING,
  description STRING,
  weight DECIMAL(10, 2)
) WITH (
 'connector' = 'kafka',
 'topic' = 'products_binlog',
 'properties.bootstrap.servers' = 'localhost:9092',
 'properties.group.id' = 'testGroup',
 'format' = 'canal-json'  -- using canal-json as the format
)
```



changelog format

如果要使用Kafka的changelog-json Format，对于程序而言，需要添加如下依赖:

```
<dependency>
  <groupId>com.alibaba.ververica</groupId>
  <artifactId>flink-format-changelog-json</artifactId>
  <version>1.0.0</version>
</dependency>
```



如果要使用Flink SQL Client，需要添加如下jar包：**flink-format-changelog-json-1.0.0.jar**，将该jar包放在Flink安装目录的lib文件夹下即可。



```
-- assuming we have a user_behavior logs
CREATE TABLE user_behavior (
    user_id BIGINT,
    item_id BIGINT,
    category_id BIGINT,
    behavior STRING,
    ts TIMESTAMP(3)
) WITH (
    'connector' = 'kafka',  -- using kafka connector
    'topic' = 'user_behavior',  -- kafka topic
    'scan.startup.mode' = 'earliest-offset',  -- reading from the beginning
    'properties.bootstrap.servers' = 'localhost:9092',  -- kafka broker address
    'format' = 'json'  -- the data format is json
);

-- we want to store the the UV aggregation result in kafka using changelog-json format
create table day_uv (
    day_str STRING,
    uv BIGINT
) WITH (
    'connector' = 'kafka',
    'topic' = 'day_uv',
    'scan.startup.mode' = 'earliest-offset',  -- reading from the beginning
    'properties.bootstrap.servers' = 'localhost:9092',  -- kafka broker address
    'format' = 'changelog-json'  -- the data format is json
);

-- write the UV results into kafka using changelog-json format
INSERT INTO day_uv
SELECT DATE_FORMAT(ts, 'yyyy-MM-dd') as date_str, count(distinct user_id) as uv
FROM user_behavior
GROUP BY DATE_FORMAT(ts, 'yyyy-MM-dd');

-- reading the changelog back again
SELECT * FROM day_uv;
```