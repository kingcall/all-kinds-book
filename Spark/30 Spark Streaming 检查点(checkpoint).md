# Spark Streaming 检查点（checkpoint）

## 什么是Checkpointing

Checkpointing可以将RDD从其依赖关系中抽出来，保存到可靠的存储系统（例如HDFS，S3等)， 即它可以将数据和元数据保存到检查指向目录中。 因此，在程序发生崩溃的时候，Spark可以恢复此数据，并从停止的任何地方开始。

Checkpointing分为两类：

- 高可用checkpointing，容错性优先。这种类型的检查点可确保数据永久存储，如存储在HDFS或其他分布式文件系统上。 这也意味着数据通常会在网络中复制，这会降低检查点的运行速度。
- 本地checkpointing，性能优先。 RDD持久保存到执行程序中的本地文件系统。 因此，数据写得更快，但本地文件系统也不是完全可靠的，一旦数据丢失，工作将无法恢复。一般用于需要定期截取且拥有较长的lineage关系的RDD，例如，GraphX。

开发人员可以是来`RDD.checkpoint()`方法来设置检查点。在使用检查点之前，必须使用`SparkContext.setCheckpointDir(directory: String)`方法设置检查点目录。

## 为什么使用Checkpointing

RDD的检查点机制就好比Hadoop将中间计算值存储到磁盘，即使计算中出现了故障，我们也可以轻松地从中恢复。通过对 RDD 启动检查点机制可以实现容错和高可用。

- 在Spark Streaming程序中，如果某些数据已经在队列中等待处理，由于某些原因我们的应用程序崩溃，当我们再次启动时，则无需再次读取这些数据，并且数据不会丢失。
- 如果我们的应用程序正在使用任何有状态操作，那么检查点是必需的，否则一旦应用程序崩溃，所有状态都将丢失。

## 哪些RDD需要使用Checkpointing

- 计算需要很长时间的
- 计算链太长的
- 依赖于太多的父RDD

## Cache、Persist和Checkpoint的区别

### cache()与persist()的区别

会被重复使用的但是不能太大的RDD需要cache。`cache()`调用了`persist()`，区别在于cache只有一个默认的缓存级别`MEMORY_ONLY`，而persist可以根据情况设置其它的缓存级别，StorageLevel类中有12种缓存级别。

cache机制是每计算出一个要cache的partition就直接将其cache到内存了。但checkpoint没有使用这种第一次计算得到就存储的方法，而是等到job结束后另外启动专门的job去完成checkpoint ，也就是说需要checkpoint的RDD会被计算两次。因此在使用`rdd.checkpoint()`的时候建议加上`rdd.cache()`，这样第二次运行的 job 就不用再去计算该rdd了，直接读取cache写磁盘。

### persist()与checkpoint()的区别

`rdd.persist(StorageLevel.DISK_ONLY)` 与 checkpoint 也有区别。前者虽然可以将RDD的partition持久化到磁盘，但该partition由blockManager管理。一旦driver program执行结束，也就是executor所在进程CoarseGrainedExecutorBackend结束了，blockManager也会相应退出，被 cache 到磁盘上的 RDD 也会被清空，整个blockManager使用的local文件夹被删除。

而checkpoint将RDD持久化到HDFS或本地文件夹，如果不被手动remove掉，是一直存在的，也就是说可以被下一个driver program使用，而cached RDD不能被其他dirver program使用。

## 建立CheckPointing示例

用sparkContext设置hdfs的checkpoint的目录。

```
scala> sc.setCheckpointDir("hdfs:/tmp/checkpoint")
```

利用上面代码建立好检查点目录后，hdfs的会出现类似下面的目录。

```
[dev@test06 ~]$ hdfs dfs -ls /tmp/checkpoint
Found 1 items
drwxr-xr-x   - hadoop supergroup          0 2019-04-30 10:50 /tmp/checkpoint/b4282eb3-cde8-489b-afda-4f1d08b9c236
```

执行检查点

```
scala> val rdd1=sc.parallelize(1 to 1000)
rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[11] at parallelize at <console>:24
scala> rdd1.checkpoint

```

这个时候在hdfs的`/tmp/checkpoint/b4282eb3-cde8-489b-afda-4f1d08b9c236`这个目录下是找不到任何数据的。但是通过collect后，这个目录就有数据了,说明checkpoint也是个transformation的算子。

```
scala> rdd1.sum
res6: Double = 500500.0

[dev@test06 ~]$ hdfs dfs -ls /tmp/checkpoint/b4282eb3-cde8-489b-afda-4f1d08b9c236
Found 1 items
drwxr-xr-x   - ccpgdev supergroup          0 2019-04-30 10:57 /tmp/checkpoint/b4282eb3-cde8-489b-afda-4f1d08b9c236/rdd-11
```

像上面说的，由于对RDD设置检查点的时候，需要对RDD进行两次计算，所以建议在设置checkpointing之前，先对rdd调用`cache()`进行缓存起来，避免重复计算同一个rdd。

```
scala> rdd1.cache()
res8: rdd1.type = ParallelCollectionRDD[11] at parallelize at <console>:24
scala> rdd1.checkpoint()
scala> rdd1.collect()
res10: Array[Int] = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81, 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100, 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116, 117, 118, 119, 120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130, 131, 132, 133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145, 146, 147, 148, 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159, 160, 161, 162, 163, 164, 165, 166, 167, 168, 169, 170, 171, 172, 173, 174, 175, 176,...
scala>
```