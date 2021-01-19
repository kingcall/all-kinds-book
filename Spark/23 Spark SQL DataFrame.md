# Spark SQL DataFrame

DataFrame是一个分布式数据集合，它被组织成命名列。从概念上讲，它相当于具有良好优化技术的关系表。

DataFrame可以从不同来源的数组构造，例如Hive表，结构化数据文件，外部数据库或现有RDD。这个API是为现代大数据和数据科学应用程序设计的，Spark SQL的DataFrame设计灵感来自Python的Pandas和R语言的DataFrame数据结构。

## DataFrame的特性

下面是一些DataFrame的一些特征：

- 在单节点集群或者大集群，处理KB到PB级别的数据。
- 支持不同的数据格式(Avro，csv，ElasticSearch和Cassandra)和存储系统(HDFS，HIVE表，mysql等)。
- Spark SQL Catalyst 优化器。
- 可以通过Spark-Core轻松地与所有大数据工具和框架集成。
- 提供Python，Java，Scala和R等语言API。

## SparkSession

SparkSession是一个入口类，用于初始化Spark SQL的功能。

以下命令用于通过spark-shell初始化SparkSession。

```
$ spark-shell
```

使用以下命令创建SQLContext。

```
scala> import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.SparkSession
scala> val spark=SparkSession
.builder()
.appName("My Spark SQL")
.getOrCreate()
19/04/25 14:40:31 WARN sql.SparkSession$Builder: Using an existing SparkSession; some configuration may not take effect.
spark: org.apache.spark.sql.SparkSession = org.apache.spark.sql.SparkSession@560465ea
scala> import spark.implicits._
import spark.implicits._
```

`spark.implicits._`主要用来隐式转换的，比如Rdd转DataFrame

## DataFrame基本操作

DataFrame为结构化数据操作提供了一个领域特定的语言（domain-specific language）。下面会提供一些DataFrame操作结构化数据的基本示例。

读取json文件并创建DataFrame，`SQLContext.read.json`方法返回的就是DataFrame。

```
scala> val dfs = spark.read.json("hdfs:/tmp/employee.json")
dfs: org.apache.spark.sql.DataFrame = [age: string, id: string ... 1 more field]
```

**注意**：要先把employee.json文件上传到hdfs的tmp目录下。
`hdfs dfs -put employee.json /tmp`

employee.json内容如下：

```
[{"id" : "1201", "name" : "satish", "age" : "25"},
{"id" : "1202", "name" : "krishna", "age" : "28"},
{"id" : "1203", "name" : "amith", "age" : "39"},
{"id" : "1204", "name" : "javed", "age" : "23"},
{"id" : "1205", "name" : "prudvi", "age" : "23"}]
```

返回数据将会以age、id、name三个字段展示。

```
dfs: org.apache.spark.sql.DataFrame = [age: string, id: string, name: string]
```

查看DataFrame数据。

```
scala> dfs.show()
+---+----+-------+
|age|  id|   name|
+---+----+-------+
| 25|1201| satish|
| 28|1202|krishna|
| 39|1203|  amith|
| 23|1204|  javed|
| 23|1205| prudvi|
+---+----+-------+
```

使用`printSchema`方法查看DataFrame的数据模式。

```
scala> dfs.printSchema()
root
 |-- age: string (nullable = true)
 |-- id: string (nullable = true)
 |-- name: string (nullable = true)
```

使用`select()`函数查看某个列的数据。

```
scala> dfs.select("name").show()
+-------+
|   name|
+-------+
| satish|
|krishna|
|  amith|
|  javed|
| prudvi|
+-------+
```

用`filter`函数查找年龄大于`23（age> 23）`的雇员。

```
scala> dfs.filter(dfs("age")>23).show()
+---+----+-------+
|age|  id|   name|
+---+----+-------+
| 25|1201| satish|
| 28|1202|krishna|
| 39|1203|  amith|
+---+----+-------+
```

使用`groupBy`方法计算同一年龄的员工人数。类似SQL里面的`group by`语句。

```
scala> dfs.groupBy("age").count().show()
+---+-----+
|age|count|
+---+-----+
| 28|    1|
| 23|    2|
| 25|    1|
| 39|    1|
+---+-----+
```