[TOC]

## Hive 执行引擎

前面我们已经搭建起了Hive 的基础环境，每次当你使用客户端的时候，你就会看到这样的一串日志,提示我们不要再使用MR 去执行hive sql 了

````log
Hive-on-MR is deprecated in Hive 2 and may not be available in the future versions. Consider using a different execution engine (i.e. spark, tez) or using Hive 1.X releases.
````



### Tez

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



### spark

