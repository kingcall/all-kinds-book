[toc]
## Source: 数据源
- Flink 在流处理和批处理上的 source ⼤概有 4 类：基于本地集合的 source、基
于⽂件的 source、基于⽹络套接字的 source、⾃定义的 source。⾃定义的 source 常⻅的有 Apache
kafka、Amazon Kinesis Streams、RabbitMQ、Twitter Streaming API、Apache NiFi 等，当然你也可
以定义⾃⼰的 source。

## KeyBy 
- 在逻辑上是基于 key对流进⾏分区，相同的Key会被分到⼀个分区（这⾥分区指的就是下游算⼦多个并⾏节点的其中⼀个）。在内部，它使⽤ hash 函数对流进⾏分区。它返回 KeyedDataStream数据流。
- 在使用 KeyBy 函数时会把 DataStream 转换成为 KeyedStream，事实上 KeyedStream 继承了 DataStream，KeyedStream 中的元素会根据用户传入的参数进行分组
- 在生产环境中使用 KeyBy 函数时要十分注意！该函数会把数据按照用户指定的key进行分组，那么相同分组的数据会被分发到一个 subtask 上进行处理，在大数据量和 key 分布不均匀的时非常容易出现数据倾斜和反压，导致任务失败。
- ,DataStream 中分区使⽤的是 KeyBy，但是 DataSet 中使⽤的是 GroupBy




## Reduce
- Reduce 返回单个的结果值，并且 reduce 操作每处理一个元素总是创建一个新值。常用的方法有 average, sum, min, max, count，使用 reduce 方法都可实现。

## Aggregations
- Aggregations 为聚合函数的总称，常见的聚合函数包括但不限于 sum、max、min 等。Aggregations 也需要指定一个 key 进行聚合
- 事实上，对于 Aggregations 函数，Flink帮助我们封装了状态数据，这些状态数据不会被清理，**所以在实际生产环境中应该尽量避免在一个无限流上使用 Aggregations。而且，对于同一个 keyedStream ，只能调用一次 Aggregation 函数**
- 之间的区别在于 max返回流中的最⼤值，但maxBy返回具有最⼤值的键， min 和minBy 同理

```
keyedStream.sum(0);
keyedStream.sum("key");
keyedStream.min(0);
keyedStream.min("key");
keyedStream.max(0);
keyedStream.max("key");
keyedStream.minBy(0);
keyedStream.minBy("key");
keyedStream.maxBy(0);
keyedStream.maxBy("key");
```

```
DataSet<Tuple2<String, Integer>> in =
// 返回 DataSet 中前 5 的元素
DataSet<Tuple2<String, Integer>> out1 = in.first(5);
// 返回分组后每个组的前 2 元素
DataSet<Tuple2<String, Integer>> out2 = in.groupBy(0)
 .first(2);
// 返回分组后每个组的前 3 元素（按照上升排序）
DataSet<Tuple2<String, Integer>> out3 = in.groupBy(0)
 .sortGroup(1, Order.ASCENDING)
 .first(3);
```