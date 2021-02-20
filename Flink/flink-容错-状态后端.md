[toc]
## StateBackend
- 状态后端负责两件事情，第一件是本地状态管理，第二件是当checkpoint 触发的时候将state 写到远程存储
![image-20210202095048042](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202095048042.png)



![image-20210202095105832](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202095105832.png)

- RocksDBStateBackend 可以存储远超过 FsStateBackend 的状态，可以避免向 FsStateBackend 那样一旦出现状态暴增会导致 OOM，但是因为将状态数据保存在 RocksDB 数据库中，吞吐量会有所下降。

### RocksDBStateBackend
- RocksDB跟上面的都略有不同，它会在本地文件系统中维护状态，state会直接写入本地rocksdb中。同时RocksDB需要配置一个远端的filesystem。
- uri（一般是HDFS），在做checkpoint的时候，会把本地的数据直接复制到filesystem中。fail over的时候从filesystem中恢复到本地。
- RocksDB克服了state受内存限制的缺点，同时又能够持久化到远端文件系统中，比较适合在生产中使用
- 许多大型Flink流应用程序的状态存储主力是RocksDB State后台。
- 与完整检查点相比，增量检查点可以显着缩短检查点时间，但代价是（可能）更长的恢复时间。
   核心思想是增量检查点仅记录对先前完成的检查点的所有更改，而不是生成状态后台的完整，自包含备份。像这样，增量检查点建立在先前的检查点上
- rocksDB是一个本机库，它直接从进程分配内存，而不是从JVM分配内存。您必须考虑分配给RocksDB的任何内存，
   通常是将TaskManagers的JVM堆大小Reduce相同的量。不这样做可能导致YARN / Mesos / etc终止JVM进程以分配比配置更多的内存。

#### RocksDB 
- RocksDB 是 Facebook 开源的 LSM 的键值存储数据库，被广泛应用于大数据系统的单机组件中。

![image-20210202095122652](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202095122652.png)

#### 存储细节

![image-20210202095136874](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210202095136874.png)

>所有存储的 key，value 均被序列化成 bytes 进行存储。


#### 源码中如何分配磁盘
```
/** Base paths for RocksDB directory, as initialized.
这里就是我们上述设置的 12 个 rocksdb local dir */
private transient File[] initializedDbBasePaths;

/** The index of the next directory to be used from {@link #initializedDbBasePaths}.
下一次要使用 dir 的 index，如果 nextDirectory = 2，
则使用 initializedDbBasePaths 中下标为 2 的那个目录做为 rocksdb 的存储目录 */
private transient int nextDirectory;

// lazyInitializeForJob 方法中， 通过这一行代码决定下一次要使用 dir 的 index，
// 根据 initializedDbBasePaths.length 生成随机数，
// 如果 initializedDbBasePaths.length = 12，生成随机数的范围为 0-11
nextDirectory = new Random().nextInt(initializedDbBasePaths.length);
```
- 源码中使用了 random 的策略来分配 dir，跟我们所看到的现象能够匹配。随机分配有小概率会出现冲突。

#### 扩展——其他策略
- 轮循策略
- 最低负载策略
- 


### FsStateBackend
- 性能更好；日常存储是在堆内存中，面临着 OOM 的风险，不支持增量 checkpoint。
- FsStateBackend将正在运行的数据保存在TaskManager的内存中。在检查点时，它将状态SNAPSHOT写入配置的文件系统和目录中的文件。
- FsStateBackend 默认使用异步SNAPSHOT，以避免在编写状态检查点时阻塞处理管道。 

#### 适用场景
- 具有大状态，长窗口，大键/值状态的作业。
- 所有高可用性设置。


###  MemoryStateBackend
- 如果没有配置其他任何内容，系统将使用MemoryStateBackend。
- 该MemoryStateBackend保存数据在内部作为Java堆的对象。键/值状态和窗口 算子包含存储值，触发器等的哈希表。checkpoint 则存储在Jobmanager 中，所以它的特点是快，但是不稳定

#### 适用场景
- 本地开发和调试
- 几乎没有状态的作业，例如仅包含一次记录函数的作业（Map，FlatMap，Filter，...）。Kafka消费者需要很少的状态

#### 不足
- 使用 MemoryStateBackend 需要增加非常多的 Heap 空间用于存储窗口内的状态数据（样本），相对于把数据放到磁盘的优点是处理性能非常好，但缺点很明显：由于 Java 对象在内存的存储效率不高，GB 级别的内存只能存储百兆级别的真实物理数据，所以会有很大的内存开销
- JVM 大堆 GC 停机时间相对较高，影响作业整体稳定，另外遇到热点事件会有 OOM 风险。

### GeminiStateBackend
- 可以通过如下方式，在 Ververica Platform 产品中指定使用 Gemini state backend
```
state.backend=org.apache.flink.runtime.state.gemini.GeminiStateBackendFactory

```

## 配置状态后台
### 代码配置
````text
MemoryStateBackend memoryStateBackend = new MemoryStateBackend(100, false);
FsStateBackend fsStateBackend = new FsStateBackend("", false);
env.setStateBackend(fsStateBackend);
env.setStateBackend(memoryStateBackend);
````

### 配置文件配置
```text
state.backend: filesystem
state.checkpoints.dir: hdfs://namenode:40010/flink/checkpoints
```

## 状态清理的目的
### 控制状态的大小
### (敏感)数据保护的要求
- 数据进行敏感处理已经成为许多应用程序的首要任务。此类使用场景的的一个典型案例，就需要仅在特定时间段内保存数据并防止其后可以再次访问该数据
## 状态清理的配置
### 什么时候重置生存时间
- 默认情况下，当状态被修改时，生存时间就会被更新。我们也可以在读操作访问状态时更新相关项的生存时间，但这样要花费额外的写操作来更新时间戳。

### 已经过期的数据是否可以访问
- 状态生存时间机制使用的是惰性策略来清除过期状态。这可能导致应用程序会尝试读取过期但尚未删除的状态。用户可以配置对这样的读取请求是否返回过期状态。无论哪种情况，过期状态都会在之后立即被删除。虽然返回已经过期的状态有利于数据可用性，但不返回过期状态更符合相关数据保护法规的要求。

### 哪种时间语义被用于定义生存时间
- 在Apache Flink 1.8.0中，用户只能根据处理时间（Processing Time）定义状态生存时间。未来的Flink版本中计划支持事件时间（Event Time）。

## 状态清理的方式
### StateTtlConfig
### 定时器进行状态清理
- 使用这种方法，将为每个状态访问注册一个清除计时器。这种方法的清理更加精准，因为状态一旦过期就会被立刻删除。但是由于计时器会与原始状态一起存储会消耗空间，开销也更大一些