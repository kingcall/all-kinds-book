# Spark 安装（集群模式）

实际生产环境一般不会用本地模式搭建Spark。生产环境一般都是集群模式。下面就给出了安装集群模式的步骤。

## 运行环境

- 操作系统 —— Spark一般都是部署在Linux上，这里用的是Ubuntu 14.04及以上版本，也可以使用CentOS，RedHat等Linux系统，本教程用的是 Ubuntu 系统。
- Spark —— Apache Spark 2.X

因为是集群模式，所以需要多个物理节点，可以使用阿里云、腾讯云。也可以在自己机器搭建虚拟机集群。

## 在主节点安装Spark

### 安装前准备

#### 修改host文件

编辑 hosts 文件，并增加下面记录

```
sudo nano /etc/hostsMASTER-IP masterSLAVE01-IP slave01SLAVE02-IP slave02
```

注意把 MASTER-IP、SLAVE01-IP、SLAVE02-IP 替换成你自己机器的IP地址。

#### 安装Java 7

```
sudo apt-get install python-software-propertiessudo add-apt-repository ppa:webupd8team/javasudo apt-get updatesudo apt-get install oracle-java7-installer
```

#### 安装Scala

```
sudo apt-get install scala
```

#### 配置SSH

安装 `openssh-server` 和 `openssh-client`

#### 生成密钥

```
ssh-keygen -t rsa -P ""
```

#### 配置无密码SSH

把 master 节点 `.ssh/id_rsa.pub`复制到 `.ssh/authorized_keys`。其他 Slave 节点跟 Master 一样的操作过程。

#### 用SSH连接测试

用SSH命令连接到其他任意主机，看看是否需要密码。如果没提示输入密码，则免密码连接配置成功。

```
ssh slave01ssh slave02
```

### 安装Spark

#### 下载Spark

可以从Spark官网下载最新版本
http://spark.apache.org/downloads.html

#### 解压tar包

```
tar xzf spark-2.0.0-bin-hadoop2.6.tgz
```

#### 安装配置

在用户 home 目录下编辑 .bashrc，并新增环境变量。

```
export JAVA_HOME=<path-of-Java-installation> (eg: /usr/lib/jvm/java-7-oracle/)export SPARK_HOME=<path-to-the-root-of-your-spark-installation> (eg: /home/dataflair/spark-2.0.0-bin-hadoop2.6/)export PATH=$PATH:$SPARK_HOME/bin
```

让环境变量生效
`source .bashrc`

编辑 spark-env.sh
`cd $SPARK_HOME/conf/`

在该目录下并没有 spark-env.sh 文件，得从 spark-env.sh.tmplate 复制一个文件，文件名修改为 spark-env.sh
`cp spark-env.sh.template spark-env.sh`

在 spark-env.sh 新增下面的环境变量

```
export JAVA_HOME=<path-of-Java-installation> (eg: /usr/lib/jvm/java-7-oracle/)export SPARK_WORKER_CORES=8
```

新增 Slave 节点
在 `$SPARK_HOME/conf/` 目录下创建 slaves 配置文件，并在该文件写入两个节点的主机名：

```
slave01slave02
```

## 在 Slave 节点安装 Spark

### 安装前准备

- 编辑hosts文件
  - 安装java 7
  - 安装Scala

这些步骤跟Master的一样

### 把Spark安装包拷贝到所有Slave节点

先压缩安装包
`tar czf spark.tar.gz spark-2.0.0-bin-hadoop2.6`

### 把压缩好的安装包拷贝到 Slave 节点

```
scp spark.tar.gz slave01:~scp spark.tar.gz slave02:~
```

> 注意：这些命令在 Master 节点执行注意：这些命令在 Master 节点执行

### 在Slave节点解压安装包

```
tar xzf spark.tar.gz
```

> 注意：该命令在Slave节点执行

到这里，Spark 已经在 Master 和 Slave 节点安装并配置完成。可以启动 Spark 集群了。

## 启动Spark集群

### 启动Spark服务

```
sbin/start-all.sh
```

> 注意：该命令在Master节点执行

### 检查服务是否启动成功

检查Master节点进程

```
$jpsMaster
```

### 检查Slave节点进程

```
$jpsWorker
```

### Spark Web UI

spark Master节点UI界面地址和端口
[http://MASTER-IP:8080/](http://master-ip:8080/)

从这里可以看到 Spark 的 Slave 节点信息，执行中的 application，集群资源等信息

spark 应用程序UI界面地址和端口
[http://MASTER-IP:4040/](http://master-ip:4040/)

## 停止Spark集群

可以在Master执行下面命令停止Spark集群
`sbin/stop-all.sh`