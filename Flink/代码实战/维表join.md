[toc]
## 维表关联
- 维表关联是离线计算或者实时计算里面常见的一种处理逻辑，常常用于字段补齐、规则过滤等
- 一般情况下维表数据放在MySql等数据库里面，对于离线计算直接通过ETL方式加载到Hive表中，然后通过sql方式关联查询即可，但是对于实时计算中Flink、SparkStreaming的表都是抽象的、虚拟的表，那么就没法使用加载方式完成。
```
一般情况下的首选方案是Flink内置的异步I/O机制，必要时还得配合使用高效的缓存（如Guava提供的LoadingCache）减少对外部数据源的请求压力
```

### 预加载维表
```
1. 用一个RichMapFunction封装整个join过程，用一个单线程的调度线程池每隔10分钟请求MySQL，拉取想要的维度表数据存入HashMap
2. 也可以预加载，缺点就是不支持更新
```
- 维度信息延迟变更：MySQL 中的维度信息随便可能在变，但是只有每分钟才会同步一次
- 该方案适用于维表数据量较小，且维表变更频率较低的场景。

### 分发维表文件
- Distributed Cache 分发文件到TaskManager 

### 广播维表
- Flink 的 broadcast 流实时消费 MQ 中数据，就可以实时读取到维表的更新，然后配置就会在 Flink 任务生效，通过这种方法及时的修改了维度信息。
- **broadcast 可以动态实时更新配置，然后影响另一个数据流的处理逻辑**。

#### 优化
- 当维度信息较大，每台机器上都存储全量维度信息导致内存压力过大时，**可以考虑进行 keyBy，这样每台节点只会存储当前 key 对应的维度信息，但是使用 keyBy 会导致所有数据都会进行 shuffle**。
- 当然上述代码需要将维度数据广播到所有实例，也是一种 shuffle，但是维度变更一般只是少量数据，成本较低，可以接受。大家在开发 Flink 任务时应该根据实际的业务场景选择最合适的方案。
- 对已经广播出去的数据，如果不需要应该进行删除，例如如果我们是对商品信息广播
```
// 更新商品的维表信息到状态中
@Override
public void processBroadcastElement(Goods goods,
                                    Context ctx,
                                    Collector<Tuple2<Order, String>> out)
        throws Exception {
    BroadcastState<Integer, String> broadcastState =
            ctx.getBroadcastState(GOODS_STATE);
    if (goods.isRemove()) {
        // 商品下架了，应该要从状态中移除，否则状态将无限增大
        broadcastState.remove(goods.getGoodsId());
    } else {
        // 商品上架，应该添加到状态中，用于关联商品信息
        broadcastState.put(goods.getGoodsId(), goods.getGoodsName());
    }
}

```

### Temporal table

### 热存储关联
- 当维度数据较大时，不能全量加载到内存中，可以实时去查询外部存储，例如 MySQL、HBase 等。这样就解决了维度信息不能全量放到内存中的问题，但是对于吞吐量较高的场景，可能与 MySQL 交互就变成了 Flink 任务的瓶颈。每来一条数据都需要进行一次同步 IO。

#### Flink异步IO
```
配合 Google Guava 的Cache 
```
- Mysql维表关联：全量加载
- Hbase维表关联：LRU策略
- Redis维表关联：实时查询

#### 优雅的使用 Cache
- guava 的 Cache 支持两种过期策略，一种是按照访问时间过期，一种是按照写入时间过期。
- 按照访问时间过期指的是：每次访问都会延长一下过期时间，假如设置的 expireAfterAccess(300, TimeUnit.MILLISECONDS)  ，即 300ms 不访问则 Cache 中的数据就会过期。每次访问，它的过期时间就会延长至 300ms 以后。如果每 200ms 访问一次，那么这条数据将永远不会过期了。所以一定要注意避坑，如果发现Cache中数据一直是旧数据，不会变成最新的数据，可以看看是不是这个原因。
- 按照写入时间过期指的是：每次写入或者修改都会延迟一下过期时间，可以设置 expireAfterWrite(300, TimeUnit.MILLISECONDS)  表示 300ms 不写入或者不修改这个 key 对应的 value，那么这一对 kv 数据就会被删除。**就算在 300ms 访问了 1 万次 Cache，300ms 过期这条数据也会被清理，这样才能保证数据被更新**

### 配置中心
