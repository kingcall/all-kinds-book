# Spark RDD 的创建方式

创建RDD有3种不同方式：

- 并行化
- 从外部存储系统
- 从其他RDD

## 并行化创建新RDD

刚开始学习Spark的时候，通常用这个方法从数据集合创建新RDD，把数据集做作为参数传入`SparkContext.parallelize()`方法即可创建出一个新RDD。这种方法的好处就是可以在Spark shell快速创建RDD，并在RDD上面执行各种操作。但是除了测试代码效果之外，在实际的开发工作中很少用这种方法，因为它要求所有的操作数据都在一台机器上。

下面是用并行化创建RDD的例子，还对创建出来的RDD进行sortByKey操作，即排序操作。

```
scala> val data=sc.parallelize(Seq(("maths",52),("english",75),("science",82), ("computer",65),("maths",85)))
data: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[0] at parallelize at <console>:24
scala> val sorted = data.sortByKey()
sorted: org.apache.spark.rdd.RDD[(String, Int)] = ShuffledRDD[3] at sortByKey at <console>:26
scala> sorted.foreach(println)
[Stage 2:>                                                          (0 + 0) / 5](computer,65)
(maths,52)
(maths,85)
(science,82)
(english,75)
scala>
```

用并行化创建RDD最关键的一点就是数据集到底被分成了多少分区。Spark集群中，一个task运行一个分区。一般要求每个CPU处理2到4个分区。Spark会基于集群设置分区数。可以用`rdd.partitions.size`查看rdd的分区数。你也可以把分区数作为第二个参数传递给parallelize来达到手动设置分区数的效果。比如手动设置10个分区：

```
sc.parallelize(data, 10)
```

手动设置分区数，其中coalesce是对结果RDD重新分区。

```
scala> val rdd1 = sc.parallelize(Array("jan","feb","mar","april","may","jun"),3)
rdd1: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[8] at parallelize at <console>:24
scala> rdd1.partitions.size
res11: Int = 3
#rdd1的分区为3个
scala> val result = rdd1.coalesce(2)
result: org.apache.spark.rdd.RDD[String] = CoalescedRDD[9] at coalesce at <console>:26
scala> result.partitions.size
res12: Int = 2
#result分区变为2个
scala> result.foreach(println)
mar
jan
feb
april
may
jun
scala>
```

## 从外部存储系统创建RDD

Spark是支持使用任何Hadoop支持的存储系统上的文件创建RDD的，比如说HDFS、Cassandra、HBase以及本地文件。通过调用SparkContext的`textFile()`方法，可以针对本地文件或HDFS文件创建RDD。

通过本地文件或HDFS创建RDD的几个注意点 ：

- 如果是针对本地文件的话：
  - 如果是在Windows上进行本地测试，windows上有一份文件即可；
  - 如果是在Spark集群上针对Linux本地文件，那么需要将文件拷贝到所有worker节点上，因为用spark-submit提交程序的时候使用—master指定了master节点，在standlone模式下，textFile()方法内仍然使用的是Linux本地文件，在这种情况下，就需要将文件拷贝到所有worker节点上，不然会报找不到文件异常。
- Spark的textFile()方法支持针对目录、压缩文件以及通配符进行RDD创建。
- Spark默认会为hdfs文件的每一个block创建一个partition，但是也可以通过textFile()的第二个参数手动设置分区数量，只能比block数量多，不能比block数量少。

### 从csv文件生成RDD

调用`csv(String path)`方法。

```
import org.apache.spark.sql.SparkSession
def main(args: Array[String]):Unit = {
object DataFormat {
val spark =  SparkSession.builder.appName("AvgAnsTime").master("local").getOrCreate()
val dataRDD = spark.read.csv("path/of/csv/file").rdd
```

这里的`.rdd`方法把`DataSet<Row>`转换成`RDD<Row>`类型。

### 从json文件生成RDD

调用`json(String Path)`方法，加载json文件并放回`DataSet<Row>`数据，一行表示一个`DataSet<Row>`对象。

```
val dataRDD = spark.read.json("path/of/json/file").rdd
```

### 从文本文件生成RDD

调用`textFile(String path)`方法。该方法将返回字符串数据集。

```
val dataRDD = spark.read.textFile("path/of/text/file").rdd
```

## 从其他RDD生成新RDD

Spark的Transformation操作将从一个给定的RDD生成新RDD。filter，count，distinct，Map，FlagMap这些都是Transformation操作。

```
scala> val words=sc.parallelize(Seq("the", "quick", "brown", "fox", "jumps", "over", "the", "lazy", "dog")) 
words: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[4] at parallelize at <console>:24
scala> val wordPair = words.map(w => (w.charAt(0), w))
wordPair: org.apache.spark.rdd.RDD[(Char, String)] = MapPartitionsRDD[5] at map at <console>:26
scala> wordPair.foreach(println)
[Stage 0:>                                                         (0 + 0) / 16](l,lazy)
(f,fox)
(d,dog)
(b,brown)
(q,quick)
(o,over)
(j,jumps)
(t,the)
(t,the)
scala>
```