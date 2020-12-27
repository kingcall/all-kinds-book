[TOC]

## Hive 执行引擎

前面我们已经搭建起了Hive 的基础环境，每次当你使用客户端的时候，你就会看到这样的一串日志,提示我们不要再使用MR 去执行hive sql 了

````log
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
````

![hive-execution](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/26/12:31:59-hive-execution.png)

## Tez

tez 是基于hive 之上，可以将sql翻译解析成DAG计算的引擎。基于DAG 与mr 架构本身的优缺点，tez 本身经过测试一般小任务在hive mr 的2-3倍速度左右，大任务7-10倍左右，根据情况不同可能不一样。



#### Tez 安装配置



http://www.apache.org/dyn/closer.lua/tez/0.9.2/

hdfs dfs -mkdir /app

dfs dfs -put Downloads/apache-tez-0.9.2-bin.tar.gz /app

```
export TEZ_HOME=/usr/local/tez/apache-tez-0.9.2-bin
export TEZ_CONF=/usr/local/tez/apache-tez-0.9.2-bin/conf

export HADOOP_HOME=/usr/local/Cellar/hadoop/3.2.1/libexec
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export HADOOP_CLASSPATH=`hadoop classpath`
export HADOOP_CLASSPATH=${HADOOP_CLASSPATH}:${TEZ_HOME}/*:${TEZ_CONF_DIR}:${TEZ_HOME}/lib/*

```



![image-20201223111303552](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/23/11:13:04-image-20201223111303552.png)

执行`hadoop classpath` 发现jar 包已经被引入进来了



#### 使用



## Spark

Hive从1.1之后，支持使用Spark作为执行引擎，配置使用Spark On Yarn作为Hive的执行引擎，首先需要注意以下两个问题：

**Hive的版本和Spark的版本要匹配；**

具体来说，你使用的Hive版本编译时候用的哪个版本的Spark，那么就需要使用相同版本的Spark，可以在Hive的pom.xml中查看spark.version来确定；

```
Hive root pom.xml’s <spark.version> defines what version of Spark it was built/tested with.
```

**Spark使用的jar包，必须是没有集成Hive的；**

也就是说，编译时候没有指定-Phive.

一般官方提供的编译好的Spark下载，都是集成了Hive的，因此这个需要另外编译。

Note that you must have a version of Spark which does **not** include the Hive jars. Meaning one which was not built with the Hive profile.

如果不注意版本问题，则会遇到各种错误，比如：

1. Caused by: java.lang.NoClassDefFoundError: org/apache/hive/spark/client/Job
2. Caused by: java.lang.ClassNotFoundException: org.apache.hive.spark.client.Job

我这里使用的环境信息如下：

````
hadoop-2.3.0-cdh5.0.0
apache-hive-2.0.0-bin
spark-1.5.0-bin-hadoop2.3
````

其中，Spark使用了另外编译的spark-assembly-1.5.0-hadoop2.3.0.jar。编译很简单，下载spark-1.5.0的源码，使用命令：

```shell
mvn -Pyarn -Phadoop-2.3 -Dhadoop.version=2.3.0-cdh5.0.0 -DskipTests -Dscala-2.10 clean package
```

首先在hive-site.xml中添加spark.home：

```xml
<property>
<name>spark.home</name>
<value>/usr/local/spark/spark-1.5.0-bin-hadoop2.3</value>
</property>
```

同时也配置了环境变量

```shell
export SPARK_HOME=/usr/local/spark/spark-1.5.0-bin-hadoop2.3
```

这两块应该只需要配置一处即可。

进入hive-cli命令行，使用set的方式设置以下参数：

```shell
set spark.master=yarn-cluster; //默认即为yarn-cluster模式，该参数可以不配置
set hive.execution.engine=spark;
set spark.eventLog.enabled=true;
set spark.eventLog.dir=hdfs://cdh5/tmp/sparkeventlog;
set spark.executor.memory=1g;
set spark.executor.instances=50; //executor数量，默认貌似只有2个
set spark.driver.memory=1g;
set spark.serializer=org.apache.spark.serializer.KryoSerializer;
```

以上参数不是必须的，这个参数必须：`set hive.execution.engine=spark;`。

当然，这些参数也可以配置在hive-site.xml中。接下来就可以执行HQL查询试试了.