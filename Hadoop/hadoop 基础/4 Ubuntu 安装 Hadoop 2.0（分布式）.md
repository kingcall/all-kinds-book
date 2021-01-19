# Ubuntu 安装 Hadoop 2.0（分布式）

## 软件版本

操作系统：Linux操作系统，Ubuntu 14.04或者16.04及以上版本。
Hadoop：Cloudera公司发布的Hadoop版本CDH5.x，你也可以使用Apache Hadoop 2.x。

## Master节点安装Hadoop

### 准备

#### 配置hosts文件

在Master节点和Slave节点的hosts文件增加以下记录。

```
sudo nano /etc/hostsMASTER-IP masterSLAVE01-IP slave01SLAVE02-IP slave02
```

注意把 MASTER-IP、SLAVE01-IP、SLAVE02-IP 替换对应节点的IP。

### 安装Java8（建议用Oracle Java）

1、安装python-software-properties
`sudo apt-get install python-software-properties`

2、新增仓库
`sudo add-apt-repository ppa:webupd8team/java`

3、更新资源列表
`sudo apt-get update`

4、安装java
`sudo apt-get install oracle-java8-installer`

### 配置SSH

1、 安装openssh-server和openssh-client
`sudo apt-get install openssh-server openssh-client`

2、生成秘钥
`ssh-keygen -t rsa -P ""`

3、 配置免密SSH
把 master 节点的 `.ssh/id_rsa.pub` 文件的内容拷贝到所有节点（包括 master 和 slave ）的 `.ssh/authorized_keys` 文件里面。

4、测试免密配置是否成功

```
ssh slave01ssh slave02
```

### 下载Hadoop

通过下面链接下载hadoop。
http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.5.0-cdh5.3.2.tar.gz

#### 解压tar包

```
tar xzf hadoop-2.5.0-cdh5.3.2.tar.gz
```

### 配置Hadoop集群

#### 配置环境变量

在用户 home 目录下，编辑`.bashrc`文件，并增加下面环境变量：

```
export HADOOP_PREFIX="/home/ubuntu/hadoop-2.5.0-cdh5.3.2"export PATH=$PATH:$HADOOP_PREFIX/binexport PATH=$PATH:$HADOOP_PREFIX/sbinexport HADOOP_MAPRED_HOME=${HADOOP_PREFIX}export HADOOP_COMMON_HOME=${HADOOP_PREFIX}export HADOOP_HDFS_HOME=${HADOOP_PREFIX}export YARN_HOME=${HADOOP_PREFIX}
```

*注意：要确保路径都是正确的。`/home/ubuntu/hadoop-2.5.0-cdh5.3.2` 这个是我的 home 目录，你可以用`cd;pwd`命令查看你自己的home目录，并把路径替换成你自己的。*

配置完之后按“Ctrl+X”保存文件。为了使环境变量生效，可以退出终端，再重新登录。

#### 检查环境变量

检查被添加到`.bashrc`文件的环境变量是否生效。

```
bashhdfs
```

如果生效，hdfs命令应该是正常的。

#### 配置hadoop-env.sh

把JAVA_HOME添加到hadoop-env.sh里面。hadoop-env.sh在$HADOOP_HOME/etc/hadoop目录下。
`export JAVA_HOME=/usr/lib/jvm/java-8-oracle/`

#### 配置core-site.xml

编辑 $HADOOP_HOME/etc/hadoop 目录下的core-site.xml文件，并增加下面配置：

```
<configuration><property><name>fs.defaultFS</name><value>hdfs://master:9000</value></property><property><name>hadoop.tmp.dir</name><value>/home/ubuntu/hdata</value></property></configuration>
```

这里的这个目录 /home/ubuntu/hdata，用于举例。请自己指定一个由写权限的目录即可。

#### 配置hdfs-site.xml

在 $HADOOP_HOME/etc/hadoop/ 目录下，编辑 hdfs-site.xml，并增加下面配置：

```
<configuration><property><name>dfs.replication</name><value>2</value></property></configuration>
```

#### 配置mapred-site.xml

在 $HADOOP_HOME/etc/hadoop 下，编辑 mapred-site.xml 配置文件，并增加以下配置：

```
<configuration><property><name>mapreduce.framework.name</name><value>yarn</value></property></configuration>
```

#### 配置yarn-site.xml

在 $HADOOP_HOME/etc/hadoop 下，编辑 yarn-site.xml 配置文件，并增加以下配置：

```
<configuration><property><name>yarn.nodemanager.aux-services</name><value>mapreduce_shuffle</value></property><property><name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name><value>org.apache.hadoop.mapred.ShuffleHandler</value></property><property><name>yarn.resourcemanager.resource-tracker.address</name><value>master:8025</value></property><property><name>yarn.resourcemanager.scheduler.address</name><value>master:8030</value></property><property><name>yarn.resourcemanager.address</name><value>master:8040</value></property></configuration>
```

#### 配置slave文件

在 $HADOOP_HOME/etc/hadoop 目录下，配置 slave 文件，并增加下面记录：

```
slave01slave02
```

## Slave节点安装Hadoop

### 准备

下面的步骤跟Master节点一样，需要在所有slave节点执行。

- 配置hosts文件
- 安装Java8

### 把master的配置好的Hadoop安装文件拷贝到所有Slave节点

1、打包配置好的hadoop安装包，注意：下面命令在Master节点执行。
`tar czf hadoop.tar.gz hadoop-2.5.0-cdh5.3.2`

2、把打包好的tar文件拷贝到所有slave节点，注意：下面命令在Master节点执行。

```
scp hadoop.tar.gz slave01:~scp hadoop.tar.gz slave02:~
```

3、在所有slave节点解压tar文件，注意：下面命令在所有slave节点执行。
`tar xzf hadoop.tar.gz`

到这里，Hadoop已经在所有的Slave节点都安装完成了。接下来就可以启动集群了。

## 启动Hadoop集群

让我们来看一下如何启动Hadoop集群？

### 格式化namenode节点

```
bin/hdfs namenode -format
```

该命令会把HDFS里面的所有数据都清空。注意：该命令要在master节点执行。

### 启动HDFS

```
sbin/start-dfs.sh
```

该命令要在master节点执行。

### 启动Yarn

```
sbin/start-yarn.sh
```

该命令要在master节点执行。

### 检查Hadoop是否启动成功

1、Master节点

```
$jpsNameNodeResourceManager
```

2、Slave节点

```
$jpsDataNodeNodeManager
```

## 关闭Hadoop集群

让我们来看一下如何关闭Hadoop集群。

### 关闭Yarn

```
sbin/stop-yarn.sh
```

该命令要在master节点执行。

### 关闭HDFS

```
sbin/stop-dfs.sh
```

该命令要在master节点执行。