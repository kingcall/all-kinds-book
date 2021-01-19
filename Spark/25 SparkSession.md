# Spark SQL 入口类 SparkSession

在开发Spark SQL时，必须先创建一个SparkSession类，SparkSession是与集群交互的基础，也是SparkSQL的编码入口。所有的操作都要用到SparkSession提供的API。

早期的版本使用的是SQLContext或者HiveContext。**spark2.0以后，建议使用的是SparkSession**。

## 创建SparkSession

```
import org.apache.spark.sql.SparkSession
val spark = SparkSession
  .builder()
  .appName("Spark SQL basic example")
  .config("spark.some.config.option", "some-value")
  .getOrCreate()
// For implicit conversions like converting RDDs to DataFrames
import spark.implicits._
```