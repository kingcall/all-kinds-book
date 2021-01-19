# Hadoop Ubuntu 安装 Hadoop 2.0（伪分布式）

本节教程主要介绍了如何在 Ubuntu 16.0.4 操作系统上安装 Hadoop 2.0 单机器集群，也叫伪分布式模式。

## 准备

### 软件版本

操作系统：Ubuntu 16.04 及以上版本，其他 Linux 系统也可以，如 CentOS，RedHat 等。
Hadoop：Cloudera 公司发布的Hadoop版本 CDH5.x，你也可以使用 Apache Hadoop 2.x。

### 操作系统安装

如果你使用的是 Windows 或者 Mac 操作系统，可以安装个虚拟机来模拟 Linux 环境。
http://data-flair.training/blogs/install-ubuntu-vmware-player/
http://data-flair.training/blogs/step-by-step-installation-of-ubuntu-on-virtual-box/

## 环境配置

在 Ubuntu 系统下安装 Hadoop 前需要配置一下系统环境。

### 安装Java8

#### 安装 python-software-properties

为了添加 Java 软件源，我们需要下载 python-software-properties。使用 apt-get 命令安装即可：

```
sudo apt-get install python-software-properties
```

#### 添加软件仓库

现在手动添加一个软件仓库，Ubuntu将会在这里安装Java。

```
sudo add-apt-repository ppa:webupd8team/java
```

#### 更新资源列表

建议定期更新资源列表，资源列表是Ubuntu可以下载和安装软件的地方。

```
sudo apt-get update
```

#### 安装Java

现在我们可以下载并安装Java了。

```
sudo apt-get install oracle-java8-installer
```

命令执行完之后，用下面命令验证是否安装成功。

```
java –version
```

### 配置SSH

SSH 用于远程登录系统。现在需要配置免密 SSH，即不需要密码就可以登录到远程的机器。免密 SSH 用于远程脚本调用，master 可以自动远程启动 slave 节点进程。

#### 安装 openssh-server 和 openssh-client

这两个是SSH工具。

```
sudo apt-get install openssh-server openssh-client
```

#### 生成秘钥

```
ssh-keygen -t rsa -P ""
```

该命令会提示输入保存秘钥的文件名，这里保持默认值即可。直接一路 enter 之后，生成的秘钥将保存在默认路径下，比如“.ssh”目录。用`ls .ssh/`检查默认路径，你可以找到两个文件，一个是保存私钥的文件 id_rsa，一个是保存公钥的 id_rsa.pub。

#### 配置免密码SSH

把 id_rsa.pub 文件的内容拷贝到authorized_keys文件里面。

```
cat $HOME/.ssh/id_rsa.pub>>$HOME/.ssh/authorized_keys
```

#### 本地检查SSH

```
ssh localhost
```

如果配置正常的话，该命令是不会提示你输入密码的。说明免密SSH配置成功。

## 安装Hadoop

### 下载Hadoop

可以通过下面链接下载到 hadoop2.0 版本：
http://archive.cloudera.com/cdh5/cdh/5/hadoop-2.5.0-cdh5.3.2.tar.gz

下载完成后，把它上传到虚拟机，并放在home目录下：

```
mv Desktop/hadoop-2.5.0-cdh5.3.2.tar.gz /home/dataflair/
```

### 解压tar包

```
tar xzf hadoop-2.5.0-cdh5.3.2.tar.gz
```

### 配置环境变量

编辑.bashrc文件，该文件是你的home目录下本身就存在的用于设置环境变量的配置文件。

```
nano .bashrc
```

添加以下配置

```
export HADOOP_PREFIX="/home/dataflair/hadoop-2.5.0-cdh5.3.2"export PATH=$PATH:$HADOOP_PREFIX/binexport PATH=$PATH:$HADOOP_PREFIX/sbinexport HADOOP_MAPRED_HOME=${HADOOP_PREFIX}export HADOOP_COMMON_HOME=${HADOOP_PREFIX}export HADOOP_HDFS_HOME=${HADOOP_PREFIX}export YARN_HOME=${HADOOP_PREFIX}
```

*注意：要确保路径都是正确的。`/home/dataflair/hadoop-2.5.0-cdh5.3.2` 这个是我的 home 目录，你可以用`cd;pwd`命令查看你自己的 home 目录，并把路径替换成你自己的。*

配置完之后按 `Ctrl+X` 保存文件。为了使环境变量生效，可以退出终端，再重新登录。

### 配置hadoop-env.sh文件

编辑配置文件 `hadoop-env.sh`，该文件位于 `$HADOOP_HOME/etc/hadoop` 目录下，并在上面配置JAVA_HOME。

```
cd hadoop-2.5.0-cdh5.3.2/cd etc/hadoopnano hadoop-env.sh
```

在该文件设置 JAVA_HOME：

```
export JAVA_HOME=/usr/lib/jvm/java-8-oracle/
```

配置完记得保存文件。
注意：*“/usr/lib/jvm/java-8-oracle/”该目录是java默认安装路径。*

### 配置core-site.xml文件

编辑`core-site.xml`文件，路径在`$HADOOP_HOME/etc/hadoop`下面。

```
cd $HADOOP_HOME/etc/hadoopnano core-site.xml
```

在`<configuration> </configuration>`里面的末尾增加下面的配置。

```
<property><name>fs.defaultFS</name><value>hdfs://localhost:9000</value></property><property><name>hadoop.tmp.dir</name><value>/home/dataflair/hdata</value></property>
```

`Ctrl+X` 保存文件。

### 配置hdfs-site.xml文件

编辑配置文件 hdfs-site.xml，路径在 $HADOOP_HOME/etc/hadoop 下面。

```
cd $HADOOP_HOME/etc/hadoopnano hdfs-site.xml
```

在`<configuration> </configuration>`里面的末尾增加如下配置并保存：

```
<property><name>dfs.replication</name><value>1</value></property>
```

### 配置mapred-site.xml文件

在`$HADOOP_HOME/etc/hadoop`目录下，需要从模板文件`mapred-site.xml.template`拷贝一个并命名为`mapred-site.xml`。

```
cd $HADOOP_HOME/etc/hadoopcp mapred-site.xml.template mapred-site.xml
```

### 编辑mapred-site.xml

```
nano mapred-site.xml
```

在文件末尾增加下面配置：

```
<property><name>mapreduce.framework.name</name><value>yarn</value></property>
```

### 配置yarn-site.xml文件

进入到`$HADOOP_HOME/etc/hadoop`目录下，编辑`yarn-site.xml`文件：

```
cd $HADOOP_HOME/etc/hadoopnano yarn-site.xml
```

在文件末尾增加如下配置：

```
<property><name>yarn.nodemanager.aux-services</name><value>mapreduce_shuffle</value></property><property><name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name><value>org.apache.hadoop.mapred.ShuffleHandler</value></property>
```

## 启动集群

### 格式化namenode节点

```
hdfs namenode -format
```

成功安装好hadoop之后，使用该命令将会把HDFS里面的数据全部删除。

### 启动HDFS服务

```
start-dfs.sh
```

### 启动Yarn服务

```
start-yarn.sh
```

### 检查在运行的Hadoop服务

```
dataflair@ubuntu:~$ jpsNameNodeDataNodeResourceManagerNodeManager
```

到这里单节点 Hadoop 集群就搭建好了。