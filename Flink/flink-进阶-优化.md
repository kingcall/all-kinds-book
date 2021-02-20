[toc]

## Flink大状态的优化
- Flink 支持多种 StateBackend，当状态比较大时目前只有 RocksDBStateBackend 可供选择。RocksDB 是基于 LSM 树原理实现的 KV 数据库，LSM 树读放大问题比较严重，因此对磁盘性能要求比较高，强烈建议生产环境使用 SSD 做为 RocksDB 的存储介质。
> 但是有些集群可能并没有配置 SSD，仅仅是普通的机械硬盘，当 Flink 任务比较大，且对状态访问比较频繁时，机械硬盘的磁盘 IO 可能成为性能瓶颈。

### 使用多块硬盘来分担压力
- RocksDB 使用内存加磁盘的方式存储数据，当状态比较大时，磁盘占用空间会比较大。如果对 RocksDB 有频繁的读取请求，那么磁盘 IO 会成为 Flink 任务瓶颈。
- 强烈建议在 flink-conf.yaml 中配置 state.backend.rocksdb.localdir 参数来指定 RocksDB 在磁盘中的存储目录。当一个 TaskManager 包含 3 个 slot 时，那么单个服务器上的三个并行度都对磁盘造成频繁读写，从而导致三个并行度的之间相互争抢同一个磁盘 io，这样务必导致三个并行度的吞吐量都会下降。
- 庆幸的是 Flink 的 state.backend.rocksdb.localdir参数可以指定多个目录，一般大数据服务器都会挂载很多块硬盘，我们期望同一个TaskManager的三个slot使用不同的硬盘从而减少资源竞争。

```
state.backend.rocksdb.localdir: /data1/flink/rocksdb,/data2/flink/rocksdb,/data3/flink/rocksdb,/data4/flink/rocksdb,/data5/flink/rocksdb,/data6/flink/rocksdb,/data7/flink/rocksdb,/data8/flink/rocksdb,/data9/flink/rocksdb,/data10/flink/rocksdb,/data11/flink/rocksdb,/data12/flink/rocksdb
```

![image-20210202091515039](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202091515039.png)

> 可以看出三个大状态算子的并行度分别对应了三块磁盘，这三块磁盘的 IO 平均使用率都保持在 45% 左右，IO 最高使用率几乎都是 100%，而其他磁盘的 IO 平均使用率为 10% 左右，相对低很多。由此可见使用RocksDB做为状态后端且有大状态的频繁读写操作时，对磁盘 IO 性能消耗确实比较大。


### 选择合适的状态后端
- 例如增量的RocksDB

### 设置合理的状态存活时间
- 不止要设置合理的状态存活时间，对于一些特殊的状态需要通过定时器来进行清除

## Flink—代码优化
### 合理的并行度何止
#### Source 端并行度的配置
#### 中间 Operator 并行度的配置
#### Sink 端并行度的配置
### Operator Chain

## Flink-SQL 的优化
## 去重优化
- set 或者 Map 来结合 Flink state 进行去重。但是这些去重方案会随着数据量不断增大，从而导致性能的急剧下降，比如刚刚我们分析过的 hash 冲突带来的写入性能问题，内存过大导致的 GC 问题，TaskManger 的失联问题。

- bitmap
- boolmfilter

## 数据倾斜
- 第一种场景是当我们的并发度设置的比分区数要低时，就会造成上面所说的消费不均匀的情况。
- 第二种提到的就是 key 分布不均匀的情况，可以通过添加随机前缀打散它们的分布，使得数据不会集中在几个 Task 中。
- 在每个节点本地对相同的 key 进行一次聚合操作，类似于 MapReduce 中的本地 combiner。map-side 预聚合之后，每个节点本地就只会有一条相同的 key，因为多条相同的 key 都被聚合起来了。其他节点在拉取所有节点上的相同 key 时，就会大大减少需要拉取的数据数量，从而也就减少了磁盘 IO 以及网络传输开销。