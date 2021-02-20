[toc]
## watermark
- 水印的出现是为了解决实时计算中的数据乱序问题，它的本质是 DataStream 中一个带有时间戳的元素。如果 Flink 系统中出现了一个 WaterMark T，那么就意味着 EventTime<T的数据都已经到达，窗口的结束时间和T相同的那个窗口被触发进行计算了
- 也就是说：水印是 Flink 判断迟到数据的标准，同时也是窗口触发的标记。

### watermark 的意义
![image-20210202090443264](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202090443264.png)

- Watermark 是一种衡量EventTime进展的机制，它是数据本身的一个隐藏属性，数据本身携带着对应的 Watermark
- Watermark本质来说就是一个时间戳，代表着比这时间戳早的事件已经全部到达窗口，即假设不会再有比这时间戳还小的事件到达，这个假设是触发窗口计算的基础，只有Watermark大于窗口对应的结束时间，窗口才会关闭和进行计算。按照这个标准去处理数据，那么如果后面还有比这时间戳更小的数据，那么就视为迟到的数据，对于这部分迟到的数据，Flink也有相应的机制去处理
- Flink的窗口处理流式数据虽然提供了基础EventTime的WaterMark机制，但是只能在一定程度上解决数据乱序问题。
- 而某些极端情况下数据延迟会非常严重，即便通过WaterMark机制也无法等到数据全部进入窗口再进行处理。
- 默认情况下，Flink会将这些严重迟到的数据丢弃掉；如果用户希望即使数据延迟到达，也能够按照流程处理并输出结果，此时可以借助Allowed Lateness机制来对迟到的数据进行额外的处理。
- flink还提供了SideOutput对延迟数据做处理，相当于最后兜底操作，所有过期延迟数据(watermark 和 Allowed Lateness都没有拯救的数据)指定窗口已经彻底关闭了，就会把数据放到侧输出流

### 传播方式
- watermark 在传播的时候有一个特点是，它的传播是幂等的。多次收到相同的
watermark，甚至收到之前的 watermark 都不会对最后的数值产生影响，因为对于
单个输入永远是取最大的，而对于整个任务永远是取一个最小的。
### 使用
- 使用这种方式周期性生成水印的话，你可以通过`env.getConfig().setAutoWatermarkInterval(...);`来设置生成水印的间隔（每隔 n 毫秒）。

- 通常建议在数据源（source）之后就进行生成水印，或者做些简单操作比如 filter/map/flatMap之后再生成水印，越早生成水印的效果会更好，也可以直接在数据源头就做生成水印。比如你可以在 source 源头类中的 run() 方法里面这样定义
- Flink 提供了 assignTimestampsAndWatermarks() 方法来实现水印的提取和指定，该方法接受的入参有 AssignerWithPeriodicWatermarks 和 AssignerWithPunctuatedWatermarks 两种
- 因为 WaterMark 的生成是以对象的形式发送到下游，同样会消耗内存，因此水印的生成时间和频率都要进行严格控制，否则会影响我们的正常作业

#### AssignerWithPunctuatedWatermarks
- 数据流中每一个递增的 EventTime 都会产生一个 Watermark。在实际的生产环境中，在 TPS 很高的情况下会产生大量的
Watermark，可能在一定程度上会对下游算子造成一定的压力所以只有在实时性要求非常高的场景才会选择这种方式来进行水印的生成。
- 这种水印的生成方式 Flink 没有提供内置实现，它适用于根据接收到的消息判断是否需要产生水印的情况，用这种水印生成的方式并不多见。

##### AscendingTimestampExtractor

##### BoundedOutOfOrdernessTimestampExtractor

##### IngestionTimeExtractor

#### AssignerWithPeriodicWatermarks
- 周期性的（一定时间间隔或者达到一定的记录条数）产生一个 Watermark。
- 在实际的生产环境中，通常这种使用较多，它会周期性产生 Watermark 的方式
- 但是必须结合时间或者积累条数两个维度，否则在极端情况下会有很大的延时，所以Watermark的生成方式需要根据业务场景的不同进行不同的选择

### 处理方式
![image-20210202090506633](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202090506633.png)
- 一个算子的实例在收到 watermark 的时候，首先要更新当前的算子时间，这样
的话在 ProcessFunction 里方法查询这个算子时间的时候，就能获取到最新的时间。
```
  override def processElement(i: Object, context: ProcessFunction[Object, Object]#Context, collector: Collector[Object]): Unit = {
   // 可以获取算子的当前时间
    context.timestamp()
    
  }
```
- 第二步它会遍历计时器队列，这个计时器队列就是我们刚刚说到的 timer，你可以同
时注册很多 timer，Flink 会把这些 Timer 按照触发时间放到一个优先队列中。第三
步 Flink 得到一个时间之后就会遍历计时器的队列，然后逐一触发用户的回调逻辑。
- 通过这种方式，Flink 的某一个任务就会将当前的 watermark 发送到下游的其他任务
实例上，从而完成整个 watermark 的传播，从而形成一个闭环

###  Kafka 分区的时间戳

### Watermark 与 Window 结合来处理延迟数据

其实在上文中已经提到的一点是在设置 Periodic Watermark
时，是允许提供一个参数，表示数据最大的延迟时间。其实这个值要结合自己的业务以及数据的情况来设置，如果该值设置的太小会导致数据因为网络或者其他的原因从而导致乱序或者延迟的数据太多，那么最后窗口触发的时候，可能窗口里面的数据量很少，那么这样计算的结果很可能误差会很大，对于有的场景（要求正确性比较高）是不太符合需求的。但是如果该值设置的太大，那么就会导致很多窗口一直在等待延迟的数据，从而一直不触发，这样首先就会导致数据的实时性降低，另外将这么多窗口的数据存在内存中，也会增加作业的内存消耗，从而可能会导致作业发生
OOM 的问题。
