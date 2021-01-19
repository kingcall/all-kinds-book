# Spark SQL 数据源

Spark SQL支持读取很多种数据源，比如parquet文件，json文件，文本文件，数据库等。下面列出了具体的一些数据源：

- Parquet文件
- Orc文件
- Json文件
- Hive表
- JDBC

先把people.json导入到hdfs的tmp目录下。people.json内容如下：

```
{"name":"Michael","age":20}
{"name":"Andy", "age":30}
{"name":"Justin", "age":19}
```

## 通用的load/save函数

spark提供了通用的`load`和`save`函数用于加载和保存数据。支持多种数据格式：json，parquet，jdbc，orc，libsvm，csv，text等。

```
scala> val peopleDF = spark.read.format("json").load("hdfs:/tmp/people.json")
peopleDF: org.apache.spark.sql.DataFrame = [age: bigint, name: string]
scala> peopleDF.select("name","age").write.format("parquet").save("/tmp/peoplenew.parquet")
```

默认的是parquet，可以通过`spark.sql.sources.default`，修改默认配置。

## Parquet文件

```
scala> val parquetFileDF=spark.read.parquet("hdfs:/tmp/peoplenew.parquet")
parquetFileDF: org.apache.spark.sql.DataFrame = [name: string, age: bigint]
scala> peopleDF.write.parquet("/tmp/people.parquet")
```

## Orc文件

```
scala> val df=spark.read.json("hdfs:/tmp/people.json")
df: org.apache.spark.sql.DataFrame = [age: bigint, name: string]
scala> df.write.mode("append").orc("/tmp/people")
```

在hdfs查看`/tmp/people`目录

```
$ hdfs dfs -ls /tmp/people
Found 2 items
-rw-r--r--   3 ccpgdev supergroup          0 2019-04-25 17:24 /tmp/people/_SUCCESS
-rw-r--r--   3 ccpgdev supergroup        343 2019-04-25 17:24 /tmp/people/part-00000-3eea0d3e-4349-4cc0-90c7-45c423069284-c000.snappy.orc
```

spark sql 读取orc文件

```
scala> spark.read.orc("/tmp/people").show()
+---+-------+
|age|   name|
+---+-------+
| 20|Michael|
| 30|   Andy|
| 19| Justin|
+---+-------+
```

## Json文件

```
scala> val df=spark.read.json("hdfs:/tmp/people.json")
df: org.apache.spark.sql.DataFrame = [age: bigint, name: string]
scala> df.write.mode("overwrite").json("/tmp/peoplejson/")
scala> spark.read.json("/tmp/peoplejson/").show()
+---+-------+
|age|   name|
+---+-------+
| 20|Michael|
| 30|   Andy|
| 19| Justin|
+---+-------+
$ hdfs dfs -ls /tmp/peoplejson
Found 2 items
-rw-r--r--   3 ccpgdev supergroup          0 2019-04-25 17:28 /tmp/peoplejson/_SUCCESS
-rw-r--r--   3 ccpgdev supergroup         80 2019-04-25 17:28 /tmp/peoplejson/part-00000-52a02853-e85b-45eb-ba66-4ab92c3f7142-c000.json
```

## Hive表

Spark 1.6及以前的版本使用hive表需要hivecontext。Spark 2.0开始只需要创建sparksession增加`enableHiveSupport()`即可。

先在Hive的tmp库下新建一张表tmp_building_num，并插入一些数据。

```
hive> create table tmp_building_num(x int);
OK
Time taken: 0.127 seconds
hive> insert into tmp_building_num values(1);
Query ID = ccpgdev_20190425174242_bff1a3ed-b02c-47e7-bb11-8a05eb5c70c1
...省略日志...
Stage-Stage-1: Map: 1   Cumulative CPU: 4.73 sec   HDFS Read: 3196 HDFS Write: 78 SUCCESS
Total MapReduce CPU Time Spent: 4 seconds 730 msec
OK
Time taken: 22.339 seconds
hive> select * from tmp_building_num;
OK
1
Time taken: 0.154 seconds, Fetched: 1 row(s)
scala> import org.apache.spark.sql.SparkSession
import org.apache.spark.sql.SparkSession
scala> val spark = SparkSession.
| builder().
| enableHiveSupport().
| getOrCreate()
19/04/25 17:38:48 WARN sql.SparkSession$Builder: Using an existing SparkSession; some configuration may not take effect.
spark: org.apache.spark.sql.SparkSession = org.apache.spark.sql.SparkSession@498f1f63
scala> spark.sql("select count(1) from tmp.tmp_building_num").show();
+--------+
|count(1)|
+--------+
|     1  |
+--------+
```

## JDBC

数据写入mysql。

```
scala> df.repartition(1).write.mode("append").option("user", "root")
 .option("password", "password").jdbc("jdbc:mysql://localhost:3306/test","alluxio",new Properties())
```

从mysql里读数据。

```
scala> val fromMysql = spark.read.option("user", "root")
 .option("password", "password").jdbc("jdbc:mysql://localhost:3306/test","alluxio",new Properties())
```