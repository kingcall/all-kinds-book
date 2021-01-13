# Spark Shell 的使用

Spark shell 作为一个强大的交互式数据分析工具，提供了一个简单的方式学习 API。它可以使用 Scala（在Java 虚拟机上运行现有的Java库的一个很好方式）或 Python。

## Scala —— Spark Shell 命令

启动 Spark Shell
`$bin/spark-shell`

在 Spark Shell 命令行下可以创建 RDD，并对 RDD 执行转换和行动操作。

## 创建RDD

创建 RDD 主要有以下三种方式：

### 从本地文件系统创建RDD

先在 Spark_Home 目录下创建 data.txt。

```
scala> val data = sc.textFile("data.txt")
```

其中，sc 是 SparkContext 对象，在启动 Spark Shell 的时候自动生成的。

如果数据已经存在外部文件系统，例如本地文件系统，HDFS，HBase，Cassandra，S3 等，可以使用这种方式，即调用 SparkContext 的 textFile 方法，并把文件目录或者路径作为参数。

### 用 Parallelize 函数创建 RDD

```
scala> val no = Array(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)scala> val noData = sc.parallelize(no)
```

这种方法可以用于数据集已经存在的情况。

### 从其他RDD创建新RDD

```
scala> val newRDD = no.map(data => (data * 2))
```

## RDD 总记录条数

计算 RDD 的总记录数可以使用 count() 函数
`scala> data.count()`

## 数据过滤操作

过滤操作可以使用 RDD 的 filter 操作，即从已经存在的 RDD 中按条件过滤并创建新 RDD。
`scala> val DFData = data.filter(line => line.contains("Elephant"))`

## 执行转换操作和行动操作

对于比较负责的数据处理需求，可以用点操作符把转换和行动操作串起来执行。比如 filter 操作和 count 操作：
`scala> data.filter(line => line.contains("Elephant")).count()`

## 读取 RDD 第一条记录

为了从文件读取第一个记录，可以使用first()函数
`scala> data.first()`

## 从 RDD 读取5条记录

```
scala> data.take(5)
```

## RDD 分区

一个 RDD 通常都会被分成多个分区，查看分区数：
`scala> data.partitions.length`

> 注意：默认情况下，RDD会有2个分区。如果从HDFS创建新RDD，那么HDFS数据文件的block数将等于分区数。

## 缓存 RDD

缓存 RDD 可以显著提高数据读取速度和计算速度。一旦把 RDD 缓存在内存中，后续使用这个 RDD 的计算，都会从内存中取数据，这样可以减少磁盘寻道时间，提高数据计算性能。

```
scala> data.cache()
```

上面这个操作其实是个转换（Tranformation）操作，也就是说这个命令执行完，RDD 并不会被立即缓存，如果你查看Spark Web UI页面：`http://localhost:4040/storage`，你是找不到相关缓存信息的。执行`cache()`操作，RDD并不会立即缓存，直到执行行动（Action）操作，数据才会真正缓存在内存中。比如`count()`或者`collect()`：

```
scala> data.count()scala> data.collect()
```

现在我们已经执行了行动操作，执行这些操作需要从磁盘读取数据，Spark在处理这些操作的时候，会把数据缓存起来，后续不管对该RDD执行转换操作还是行动操作，都将直接从内存读取，而不需要和磁盘进行交互。所以，可以把多次使用的RDD缓存起来，提升数据处理性能。

## 从 HDFS 读取数据

要读取 HDFS 的文件，必须要提供文件的完整 URL。且文件系统标识是 hdfs，比如：
hdfs://IP:PORT/PATH
`scala> var hFile = sc.textFile("hdfs://localhost:9000/inp")`

## 用 Scala 编写 wordcout 应用

wordcount 应用，即英文单词数量统计应用，堪称大数据界的 hello word 程序。是最经典的 MapReduce 操作案例之一。

```
scala> val wc = hFile.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey(_ + _)
```

在控制台读取前5个统计结果：

```
scala> wc.take(5)
```

## 把计算结果写入HDFS文件

可以用 saveAsTextFile 操作把计算好的结果保存在 HDFS。

```
scala> wc.saveAsTextFile("hdfs://localhost:9000/out")
```