[TOC]

## Hue

HUE是一个开源的Apache Hadoop UI系统，早期由Cloudera开发，后来贡献给开源社区。HUE=**Hadoop User Experience** 它是基于Python Web框架Django实现的

通过使用Hue我们可以在浏览器端的Web控制台上与Hadoop集群进行交互来分析处理数据，例如操作HDFS上的数据，运行MapReduce Job，执行Hive的SQL语句，浏览HBase数据库等等。

官网给我们提供[测试站点](https://demo.gethue.com/hue)，你可以先睹为快，然后再决定是否要学习

### 主要初识(主要功能)



#### SQL 编辑器

SQL编辑器，支持Hive, Impala, MySQL, Oracle, PostgreSQL, SparkSQL, Solr SQL, Phoenix…

![Image](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/20/21:41:21-hue_4_assistant_2-20201220214120808.gif)

Hue brings the best [Querying Experience](https://docs.gethue.com/user/querying/) with the most intelligent autocompletes, query sharing, result charting and download for any database. Enable more of your employees to level-up and perform self service analytics like [Customer 360s](https://gethue.com/self-service-bi-doing-a-customer-360-by-querying-and-joining-salesforce-marketing-and-log-datasets/) .

Pick one of the multiple interpreters for [Apache Hive](https://docs.gethue.com/administrator/configuration/connectors/#apache-hive), [Apache Impala](https://docs.gethue.com/administrator/configuration/connectors/#apache-impala), [Presto](https://docs.gethue.com/administrator/configuration/connectors/#presto) and all the others too: MySQL, [Apache Flink SQL](https://gethue.com/blog/tutorial-query-live-data-stream-with-flink-sql/), Oracle, [SparkSQL](https://docs.gethue.com/administrator/configuration/connectors/#apache-spark-sql), [Apache Phoenix](https://gethue.com/sql-querying-apache-hbase-with-apache-phoenix/), [ksqlDB](https://gethue.com/blog/tutorial-query-live-data-stream-with-kafka-sql/), [Elastic Search](https://docs.gethue.com/administrator/configuration/connectors/#elastic-search), [Apache Druid](https://gethue.com/quick-task-how-to-query-apache-druid-analytic-database/), PostgreSQL, Redshift, BigQuery...

![Image](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/20/21:42:57-databases.svg)



1. 搜索引擎Solr的各种图表
2. Spark和Hadoop的友好界面支持
3. 支持调度系统Apache Oozie，可进行workflow的编辑、查看

**支持明细**

默认基于轻量级sqlite数据库管理会话数据，用户认证和授权，可以自定义为MySQL、Postgresql，以及Oracle
基于文件浏览器（File Browser）访问HDFS
基于Hive编辑器来开发和运行Hive查询
支持基于Solr进行搜索的应用，并提供可视化的数据视图，以及仪表板（Dashboard）
支持基于Impala的应用进行交互式查询
支持Spark编辑器和仪表板（Dashboard）
支持Pig编辑器，并能够提交脚本任务
支持Oozie编辑器，可以通过仪表板提交和监控Workflow、Coordinator和Bundle
支持HBase浏览器，能够可视化数据、查询数据、修改HBase表
支持Metastore浏览器，可以访问Hive的元数据，以及HCatalog
支持Job浏览器，能够访问MapReduce Job（MR1/MR2-YARN）
支持Job设计器，能够创建MapReduce/Streaming/Java Job
支持Sqoop 2编辑器和仪表板（Dashboard）
支持ZooKeeper浏览器和编辑器
支持MySql、PostGresql、Sqlite和Oracle数据库查询编辑器



### 快速尝鲜

因为这个软件是基于Python 开发的，所以你自己本地装的话，需要一定的环境，在公司里面一般都是大数据运维已经安装好的，我这里就不在本地安了，我们直接使用docker 进行安装，一句命令就搞定了`docker run -it -p 8888:8888 gethue/hue:latest`



### 快速定制

