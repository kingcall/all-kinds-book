# Impala 与Hive

2018-04-04阅读 6280

Impala 与Hive都是构建在Hadoop之上的数据查询工具，但是各有不同侧重，那么我们为什么要同时使用这两个工具呢?单独使用Hive或者Impala不可以吗? 一、介绍Impala和Hive (1)Impala和Hive都是提供对HDFS/Hbase数据进行SQL查询的工具，Hive会转换成MapReduce，借助于YARN进行调度从而实现对HDFS的数据的访问，而Impala直接对HDFS进行数据查询。但是他们都是提供如下的标准SQL语句，在机身里运行。

![img](https://ask.qcloudimg.com/http-save/yehe-1558124/vhg8zzahsr.png?imageView2/2/w/1620)

(2)Apache Hive是MapReduce的高级抽象，使用HiveQL,Hive可以生成运行在Hadoop集群的MapReduce或Spark作业。Hive最初由Facebook大约在2007年开发,现在是Apache的开源项目。 Apache Impala是高性能的专用SQL引擎，使用Impala SQL，因为Impala无需借助任何的框架，直接实现对数据块的查询，所以查询延迟毫秒级。Impala受到Google的Dremel项目启发，2012年由Cloudera开发，现在是Apache开源项目。 二、Impala和Hive有什么不同? (1)Hive有很多的特性： 1、对复杂数据类型(比如arrays和maps)和窗口分析更广泛的支持 2、高扩展性 3、通常用于批处理 (2)Impala更快 1、专业的SQL引擎，提供了5x到50x更好的性能 2、理想的交互式查询和数据分析工具 3、更多的特性正在添加进来 三、高级概述：

![img](https://ask.qcloudimg.com/http-save/yehe-1558124/rkl2ng7xnt.png?imageView2/2/w/1620)

 四、为什么要使用Hive和Impala? 1、为数据分析人员带来了海量数据分析能力,不需要软件开发经验,运用已掌握的SQL知识进行数据的分析。 2、比直接写MapReduce或Spark具有更好的生产力，5行HiveQL/Impala SQL等同于200行或更多的Java代码。 3、提供了与其他系统良好的互操作性，比如通过Java和外部脚本扩展，而且很多商业智能工具支持Hive和Impala。 

五、Hive和Impala使用案例 (1)日志文件分析 日志是普遍的数据类型，是当下大数据时代重要的数据源，结构不固定，可以通过Flume和kafka将日志采集放到HDFS，然后分析日志的结构，根据日志的分隔符去建立一个表，接下来运用Hive和Impala 进行数据的分析。例如：

![img](https://ask.qcloudimg.com/http-save/yehe-1558124/5n4tfr9txg.png?imageView2/2/w/1620)

 (2)情感分析 很多组织使用Hive或Impala来分析社交媒体覆盖情况。例如： 

![img](https://ask.qcloudimg.com/http-save/yehe-1558124/xbb2adfgh4.png?imageView2/2/w/1620)

 (3)商业智能 很多领先的BI工具支持Hive和Impala 

![img](https://ask.qcloudimg.com/http-save/yehe-1558124/cqu96u3ln0.png?imageView2/2/w/1620)