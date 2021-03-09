topN的常见应用场景，最热商品购买量，最高人气作者的阅读量等等。



## 1. 用到的知识点

- Flink创建kafka数据源；
- 基于 EventTime 处理，如何指定 Watermark；
- Flink中的Window，滚动（tumbling）窗口与滑动（sliding）窗口；
- State状态的使用；
- ProcessFunction 实现 TopN 功能；



## 2. 案例介绍

通过用户访问日志，计算最近一段时间平台最活跃的几位用户topN。

- 创建kafka生产者，发送测试数据到kafka；
- 消费kafka数据，使用滑动（sliding）窗口，每隔一段时间更新一次排名；



## 3. 数据源

这里使用kafka api发送测试数据到kafka，代码如下：

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class User {

    private long id;
    private String username;
    private String password;
    private long timestamp;
}

Map<String, String> config = Configuration.initConfig("commons.xml");

@Test
public void sendData() throws InterruptedException {
    int cnt = 0;

    while (cnt < 200){
        User user = new User();
        user.setId(cnt);
        user.setUsername("username" + new Random().nextInt((cnt % 5) + 2));
        user.setPassword("password" + cnt);
        user.setTimestamp(System.currentTimeMillis());
        Future<RecordMetadata> future = KafkaUtil.sendDataToKafka(config.get("kafka-topic"), String.valueOf(cnt), JSON.toJSONString(user));
        while (!future.isDone()){
            Thread.sleep(100);
        }
        try {
            RecordMetadata recordMetadata = future.get();
            System.out.println(recordMetadata.offset());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
        System.out.println("发送消息：" + cnt + "******" + user.toString());
        cnt = cnt + 1;
    }
}
```

这里通过随机数来扰乱username，便于使用户名大小不一，让结果更加明显。KafkaUtil是自己写的一个kafka工具类，代码很简单，主要是平时做测试方便。



## 4. 主要程序

创建一个main程序，开始编写代码。

#### 创建flink环境，关联kafka数据源。

```java
Map<String, String> config = Configuration.initConfig("commons.xml");

Properties kafkaProps = new Properties();
kafkaProps.setProperty("zookeeper.connect", config.get("kafka-zookeeper"));
kafkaProps.setProperty("bootstrap.servers", config.get("kafka-ipport"));
kafkaProps.setProperty("group.id", config.get("kafka-groupid"));

StreamExecutionEnvironment senv = StreamExecutionEnvironment.getExecutionEnvironment();
```

### EventTime 与 Watermark

```java
senv.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
```

设置属性`senv.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)`，表示按照数据时间字段来处理，默认是`TimeCharacteristic.ProcessingTime`

```java
/** The time characteristic that is used if none other is set. */
	private static final TimeCharacteristic DEFAULT_TIME_CHARACTERISTIC = TimeCharacteristic.ProcessingTime;
```

这个属性必须设置，否则后面，可能窗口结束无法触发，导致结果无法输出。取值有三种：

- ProcessingTime：事件被处理的时间。也就是由flink集群机器的系统时间来决定。
- EventTime：事件发生的时间。一般就是数据本身携带的时间。
- IngestionTime：摄入时间，数据进入flink流的时间，跟ProcessingTime还是有区别的；

指定好使用数据的实际时间来处理，接下来需要指定flink程序如何get到数据的时间字段，这里使用调用DataStream的assignTimestampsAndWatermarks方法，抽取时间和设置watermark。

```java
senv.addSource(
        new FlinkKafkaConsumer010<>(
                config.get("kafka-topic"),
                new SimpleStringSchema(),
                kafkaProps
        )
).map(x ->{
    return JSON.parseObject(x, User.class);
}).assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<User>(Time.milliseconds(1000)) {
    @Override
    public long extractTimestamp(User element) {
        return element.getTimestamp();
    }
})
```

前面给出的代码中可以看出，由于发送到kafka的时候，将User对象转换为json字符串了，这里使用的是fastjson，接收过来可以转化为JsonObject来处理，我这里还是将其转化为User对象`JSON.parseObject(x, User.class)`，便于处理。

这里考虑到数据可能乱序，使用了可以处理乱序的抽象类`BoundedOutOfOrdernessTimestampExtractor`，并且实现了唯一的一个没有实现的方法`extractTimestamp`，乱序数据，会导致数据延迟，在构造方法中传入了一个`Time.milliseconds(1000)`，表明数据可以延迟一秒钟。比如说，如果窗口长度是10s，



