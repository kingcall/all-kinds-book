# Spark RDD 依赖关系图

当我们从其他RDD创建RDD的时候。新的RDD就与其父RDD建立了依赖关系。这种依赖关系以图数据结构的形式存储，我们把这种依赖关系图称为**RDD的血缘关系图**。

为了更好的理解RDD之间的依赖关系图，下面是通过Cartesian和zip函数生成RDD依赖关系图。

![spark rdd 依赖关系图](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1571151305000_20191015225509312932.png)

这个依赖关系图生成过程如下：

```
val r00 = sc.parallelize(0 to 9)
val r01 = sc.parallelize(0 to 90 by 10)
val r10 = r00 cartesian r01
val r11 = r00.map(n => (n, n))
val r12 = r00 zip r01
val r13 = r01.keyBy(_ / 20)
val r20 = Seq(r11, r12, r13).foldLeft(r10)(_ union _)
```

我们可以执行`toDebugString`打印RDD的依赖关系：

```
scala> r00.toDebugString
res5: String = (20) ParallelCollectionRDD[0] at parallelize at <console>:27 []
scala> r01.toDebugString
res6: String = (20) ParallelCollectionRDD[1] at parallelize at <console>:27 []
scala> r12.toDebugString
res9: String =
(20) ZippedPartitionsRDD2[4] at zip at <console>:31 []
 | ParallelCollectionRDD[0] at parallelize at <console>:27 []
 | ParallelCollectionRDD[1] at parallelize at <console>:27 []
scala> r13.toDebugString
res10: String =
(20) MapPartitionsRDD[5] at keyBy at <console>:29 []
 | ParallelCollectionRDD[1] at parallelize at <console>:27 []
scala> r20.toDebugString
res11: String =
(460) UnionRDD[8] at union at <console>:39 []
  | UnionRDD[7] at union at <console>:39 []
  | UnionRDD[6] at union at <console>:39 []
  | CartesianRDD[2] at cartesian at <console>:31 []
  | ParallelCollectionRDD[0] at parallelize at <console>:27 []
  | ParallelCollectionRDD[1] at parallelize at <console>:27 []
  | MapPartitionsRDD[3] at map at <console>:29 []
  | ParallelCollectionRDD[0] at parallelize at <console>:27 []
  | ZippedPartitionsRDD2[4] at zip at <console>:31 []
  | ParallelCollectionRDD[0] at parallelize at <console>:27 []
  | ParallelCollectionRDD[1] at parallelize at <console>:27 []
  | MapPartitionsRDD[5] at keyBy at <console>:29 []
  | ParallelCollectionRDD[1] at parallelize at <console>:27 []
```

## toDebugString

函数原型：

```
def toDebugString: String
```

通过此函数可以获取RDD的Lineage打印输出。

## 设置打印Lineage

参数：`spark.logLineage`
默认值：`false`
设置为true时，会在执行中打印出RDD的Lineage。