# Spark SQL DataSet

**Dataset**是从Spark 1.6开始引入的一个新的抽象，当时还是处于alpha版本；然而在Spark 2.0，它已经变成了稳定版了。Dataset是特定域对象中的强类型集合，它可以使用函数或者相关操作并行地进行转换等操作。每个Dataset都有一个称为DataFrame的非类型化的视图，这个视图是行的数据集。

**DataSet和RDD主要的区别是：DataSet是特定域的对象集合；然而RDD是任何对象的集合。DataSet的API总是强类型的；而且可以利用这些模式进行优化，然而RDD却不行。**

DataFrame和Dataset是有什么关系呢？其实DataFrame是特殊的Dataset，它在编译时不会对模式进行检测。在未来版本的Spark，Dataset将会替代RDD成为我们开发编程使用的API（**注意，RDD并不是会被取消，而是会作为底层的API提供给用户使用**）。

## 如何创建DataSet

那么如何创建DataSet呢？下面给出具体代码示例。

```
scala> case class Person(name:String, age:Long)
defined class Person
scala> val caseClassDS = Seq(Person("Andy",32)).toDS()
caseClassDS: org.apache.spark.sql.Dataset[Person] = [name: string, age: bigint]
scala> caseClassDS.show()
 +----+---+
 |name|age|
+----+---+
|Andy| 32|
+----+---+
scala> val primitiveDS = Seq(1,2,3).toDS()
primitiveDS: org.apache.spark.sql.Dataset[Int] = [value: int]
scala> primitiveDS.map(_ + 1).collect()
res1: Array[Int] = Array(2, 3, 4)
scala> val path="hdfs:/tmp/people.json"
path: String = hdfs:/tmp/people.json
DataFrame转换成DataSet
scala> val peopleDS=spark.read.json(path).as[Person]
peopleDS: org.apache.spark.sql.Dataset[Person] = [age: bigint, name: string]
scala> peopleDS.show()
+---+-------+
|age|   name|
+---+-------+
| 20|Michael|
| 30|   Andy|
| 19| Justin|
+---+-------+
```