# Spark RDD 缓存机制

Spark RDD 缓存是在内存存储RDD计算结果的一种优化技术。把中间结果缓存起来以便在需要的时候重复使用，这样才能有效减轻计算压力，提升运算性能。

当对RDD执行持久化操作时，每个节点都会将自己操作的RDD的partition持久化到内存中，并且在之后对该RDD的反复使用中，直接使用内存缓存的partition。这样的话，对于针对一个RDD反复执行多个操作的场景，就只要对RDD计算一次即可，后面直接使用该RDD，而不需要反复计算多次该RDD。

巧妙使用RDD持久化，甚至在某些场景下，可以将spark应用程序的性能提升10倍。对于迭代式算法和快速交互式应用来说，RDD持久化，是非常重要的。

## 如何持久化

要持久化一个RDD，只要调用其`cache()`或者`persist()`方法即可。在该RDD第一次被计算出来时，就会直接缓存在每个节点中。而且Spark的持久化机制还是自动容错的，如果持久化的RDD的任何partition丢失了，那么Spark会自动通过其源RDD，使用transformation操作重新计算该partition。

`cache()`和`persist()`的区别在于，`cache()`是`persist()`的一种简化方式，`cache()`的底层就是调用的`persist()`的无参版本，同时就是调用`persist(MEMORY_ONLY)`，将数据持久化到内存中。如果需要从内存中去除缓存，那么可以使用`unpersist()`方法。

## RDD持久化存储级别

RDD存储级别主要有以下几种。

| 级别                  | 使用空间 | CPU时间 | 是否在内存中 | 是否在磁盘上 | 备注                                 |
| --------------------- | -------- | ------- | ------------ | ------------ | ------------------------------------ |
| MEMORY_ONLY           | 高       | 低      | 是           | 否           |                                      |
| MEMORY_ONLY_2         | 高       | 低      | 是           | 否           | 数据存2份                            |
| MEMORY_ONLY_SER       | 低       | 高      | 是           | 否           | 数据序列化                           |
| MEMORY_ONLY_SER_2     | 低       | 高      | 是           | 否           | 数据序列化，数据存2份                |
| MEMORY_AND_DISK       | 高       | 中等    | 部分         | 部分         | 如果数据在内存中放不下，则溢写到磁盘 |
| MEMORY_AND_DISK_2     | 高       | 中等    | 部分         | 部分         | 数据存2份                            |
| MEMORY_AND_DISK_SER   | 低       | 高      | 部分         | 部分         |                                      |
| MEMORY_AND_DISK_SER_2 | 低       | 高      | 部分         | 部分         | 数据存2份                            |
| DISK_ONLY             | 低       | 高      | 否           | 是           |                                      |
| DISK_ONLY_2           | 低       | 高      | 否           | 是           | 数据存2份                            |
| OFF_HEAP              |          |         |              |              |                                      |

**注意**：只能设置一种：不然会抛异常：
`Cannot change storage level of an RDD after it was already assigned a level`

异常源码如下：

```
 private def persist(newLevel: StorageLevel, allowOverride: Boolean): this.type = {    // TODO: Handle changes of StorageLevel    if (storageLevel != StorageLevel.NONE && newLevel != storageLevel && !allowOverride) {      throw new UnsupportedOperationException(        "Cannot change storage level of an RDD after it was already assigned a level")    }    // If this is the first time this RDD is marked for persisting, register it    // with the SparkContext for cleanups and accounting. Do this only once.    if (storageLevel == StorageLevel.NONE) {      sc.cleaner.foreach(_.registerRDDForCleanup(this))      sc.persistRDD(this)    }    storageLevel = newLevel    this  }
```

### MEMORY_ONLY

使用未序列化的Java对象格式，将数据保存在内存中。如果内存不够存放所有的数据，则数据可能就不会进行持久化。那么下次对这个RDD执行算子操作时，那些没有被持久化的数据，需要从源头处重新计算一遍。这是默认的持久化策略，使用cache()方法时，实际就是使用的这种持久化策略。

### MEMORY_ONLY_SER

基本含义同MEMORY_ONLY。唯一的区别是，会将RDD中的数据进行序列化，RDD的每个partition会被序列化成一个字节数组。这种方式更加节省内存，从而可以避免持久化的数据占用过多内存导致频繁GC。

### MEMORY_AND_DISK

使用未序列化的Java对象格式，优先尝试将数据保存在内存中。如果内存不够存放所有的数据，会将数据写入磁盘文件中，下次对这个RDD执行算子时，持久化在磁盘文件中的数据会被读取出来使用。

### MEMORY_AND_DISK_SER

基本含义同MEMORY_AND_DISK。唯一的区别是，会将RDD中的数据进行序列化，RDD的每个partition会被序列化成一个字节数组。这种方式更加节省内存，从而可以避免持久化的数据占用过多内存导致频繁GC。

### DISK_ONLY

使用未序列化的Java对象格式，将数据全部写入磁盘文件中。

### OFF_HEAP

这个目前是试验型选项，类似MEMORY_ONLY_SER，但是数据是存储在堆外内存的。

### 后缀带“_2”的存储级别

对于上述任意一种持久化策略，如果加上后缀_2，代表的是将每个持久化的数据，都复制一份副本，并将副本保存到其他节点上。这种基于副本的持久化机制主要用于进行容错。假如某个节点挂掉了，节点的内存或磁盘中的持久化数据丢失了，那么后续对RDD计算时还可以使用该数据在其他节点上的副本。如果没有副本的话，就只能将这些数据从源头处重新计算一遍了。

### 如何选择一种最合适的持久化策略

- 默认情况下，性能最高的当然是MEMORY_ONLY，但前提是你的内存必须足够足够大，可以绰绰有余地存放下整个RDD的所有数据。因为不进行序列化与反序列化操作，就避免了这部分的性能开销；对这个RDD的后续算子操作，都是基于纯内存中的数据的操作，不需要从磁盘文件中读取数据，性能也很高；而且不需要复制一份数据副本，并远程传送到其他节点上。但是这里必须要注意的是，在实际的生产环境中，恐怕能够直接用这种策略的场景还是有限的，如果RDD中数据比较多时（比如几十亿），直接用这种持久化级别，会导致JVM的OOM内存溢出异常。
- 如果使用MEMORY_ONLY级别时发生了内存溢出，那么建议尝试使用MEMORY_ONLY_SER级别。该级别会将RDD数据序列化后再保存在内存中，此时每个partition仅仅是一个字节数组而已，大大减少了对象数量，并降低了内存占用。这种级别比MEMORY_ONLY多出来的性能开销，主要就是序列化与反序列化的开销。但是后续算子可以基于纯内存进行操作，因此性能总体还是比较高的。此外，可能发生的问题同上，如果RDD中的数据量过多的话，还是可能会导致OOM内存溢出的异常。
- 如果纯内存的级别都无法使用，那么建议使用MEMORY_AND_DISK_SER策略，而不是MEMORY_AND_DISK策略。因为既然到了这一步，就说明RDD的数据量很大，内存无法完全放下。序列化后的数据比较少，可以节省内存和磁盘的空间开销。同时该策略会优先尽量尝试将数据缓存在内存中，内存缓存不下才会写入磁盘。
- 通常不建议使用DISK_ONLY和后缀为_2的级别：因为完全基于磁盘文件进行数据的读写，会导致性能急剧降低，有时还不如重新计算一次所有RDD。后缀为_2的级别，必须将所有数据都复制一份副本，并发送到其他节点上，数据复制以及网络传输会导致较大的性能开销，除非是要求作业的高可用性，否则不建议使用。

## 如何使用 Spark rdd 缓存

### 调用rdd.persist()

变量可以这样设置，如：`rdd.persist(StorageLevel.MEMORY_ONLY)`；这里使用了MEMORY_ONLY级别存储。当然也可以选择其他的如：`rdd.persist(StorageLevel.DISK_ONLY())`。

### 调用rdd.cache()

`cache()`是`rdd.persist(StorageLevel.MEMORY_ONLY)`的简写，效果和他一模一样的。

### 调用rdd.unpersist()清除缓存

`rdd.unpersist()`把缓存起来的RDD清除，后续如果用到该RDD，则需要重新计算。