# Spark SQL 教程

Spark为结构化数据处理引入了一个称为Spark SQL的编程模块。它提供了一个称为DataFrame的编程抽象，并且可以充当分布式SQL查询引擎。

## Spark SQL的特性

以下是Spark SQL的功能：

### 集成

无缝地将SQL查询与Spark程序混合。 Spark SQL允许您将结构化数据作为Spark中的分布式数据集(RDD)进行查询，在Python，Scala和Java中集成了API。这种紧密的集成使得可以轻松地运行SQL查询以及复杂的分析算法。

### 统一数据访问

加载和查询来自各种来源的数据。 Schema-RDDs提供了一个有效处理结构化数据的单一接口，包括Apache Hive表，镶木地板文件和JSON文件。

### Hive兼容性

在现有仓库上运行未修改的Hive查询。 Spark SQL重用了Hive前端和MetaStore，为您提供与现有Hive数据，查询和UDF的完全兼容性。只需将其与Hive一起安装即可。

### 标准连接

通过JDBC或ODBC连接。 Spark SQL包括具有行业标准JDBC和ODBC连接的服务器模式。

### 可扩展性

对于交互式查询和长查询使用相同的引擎。 Spark SQL利用RDD模型来支持中查询容错，使其能够扩展到大型作业。不要担心为历史数据使用不同的引擎。

## Spark SQL架构

下图说明了Spark SQL的体系结构

![spark sql 架构图](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1571236023000_20191016222706153277.png)

此架构包含三个层，即 **Language API**，**Schema RDD**和**数据源**。

### 语言API

Spark与不同的语言和Spark SQL兼容。 它也是由这些语言支持的API（python，scala，java，HiveQL）。

### 模式RDD

Spark Core是使用称为RDD的特殊数据结构设计的。 通常，Spark SQL适用于模式，表和记录。 因此，我们可以使用Schema RDD作为临时表。 我们可以将此Schema RDD称为数据帧。

### 数据源

通常spark-core的数据源是文本文件，Avro文件等。但是，Spark SQL的数据源不同。 这些是Parquet文件，JSON文档，HIVE表和Cassandra数据库。