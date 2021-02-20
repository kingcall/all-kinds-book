## Flink 环境准备

通过前面几篇文章，相信你已经对 Flink 的基础概念等知识已经有一定了解，现在是不是迫切的想把 Flink给用起来？先别急，我们先把电脑的准备环境给安装好，这样后面才能更愉快地玩耍。

废话不多说了，直奔主题。因为后面可能用到的有：Kafka、MySQL、ElasticSearch 等，另外像 Flink 编写程序还需要依赖Java，还有就是我们项目是用 Maven来管理依赖的，所以这篇文章我们先来安装下这个几个，准备好本地的环境，后面如果还要安装其他的组件我们到时在新文章中补充，如果你的操作系统已经中已经安装过JDK、Maven、MySQL、IDEA 等，那么你可以跳过对应的内容，直接看你未安装过的。

这里我再说下我自己电脑的系统环境：macOS High Sierra 10.13.5，后面文章的演示环境不作特别说明的话就是都在这个系统环境中。

### JDK 安装与配置

虽然现在 JDK 已经更新到 12 了，但是为了稳定我们还是安装 JDK8，如果没有安装过的话，可以去[官网](https://www.oracle.com/technetwork/java/javase/downloads/index.html)的[下载页面](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载对应自己操作系统的最新JDK8 就行。

Mac 系统的是 jdk-8u211-macosx-x64.dmg 格式、Linux 系统的是 jdk-8u211-linux-x64.tar.gz 格式。

Mac 系统安装的话直接双击然后一直按照提示就行了，最后 JDK 的安装目录在 `/Library/Java/JavaVirtualMachines/`，然后在 `/etc/hosts` 中配置好环境变量（注意：替换你自己电脑本地的路径）。


​    
    export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_152.jdk/Contents/Home
    export CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:
    export PATH=$PATH:$JAVA_HOME/bin


Linux 系统的话就是在某个目录下直接解压就行了，然后在 `/etc/profile` 添加一下上面的环境变量（注意：替换你自己电脑的路径）。然后执行 `java -version` 命令可以查看是否安装成功！

     zhisheng@zhisheng ~  java -version
    java version "1.8.0_152"
    Java(TM) SE Runtime Environment (build 1.8.0_152-b16)
    Java HotSpot(TM) 64-Bit Server VM (build 25.152-b16, mixed mode)


### Maven 安装与配置

安装好 JDK 后我们就可以安装 Maven了，我们在[官网](http://maven.apache.org/download.cgi)下载二进制包就行，然后在自己本地软件安装目录解压压缩包就行。接下来你需要配置一下环境变量：

    export M2_HOME=/Users/zhisheng/Documents/maven-3.5.2
    export PATH=$PATH:$M2_HOME/bin


然后执行命令 `mvn -v` 可以验证是否安装成功，结果如下：

    zhisheng@zhisheng ~ /Users  mvn -v
    Apache Maven 3.5.2 (138edd61fd100ec658bfa2d307c43b76940a5d7d; 2017-10-18T15:58:13+08:00)
    Maven home: /Users/zhisheng/Documents/maven-3.5.2
    Java version: 1.8.0_152, vendor: Oracle Corporation
    Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_152.jdk/Contents/Home/jre
    Default locale: zh_CN, platform encoding: UTF-8
    OS name: "mac os x", version: "10.13.5", arch: "x86_64", family: "mac"


### IDE 安装与配置

安装完 JDK 和 Maven 后，就可以安装 IDE 了，大家可以选择你熟练的 IDE 就行，我后面演示的代码都是在 IDEA中运行的，如果想为了后面不出其他的 问题的话，建议尽量和我的环境保持一致。

IDEA 官网下载地址：[下载页面的地址](https://www.jetbrains.com/idea/download/#section=mac)

下载后可以双击后然后按照提示一步步安装，安装完成后需要在 IDEA 中配置 JDK 路径和 Maven 的路径，后面我们开发也都是靠 Maven来管理项目的依赖。

### MySQL 安装与配置

因为后面文章有用到 MySQL，所以这里也讲一下如何安装与配置，首先去官网下载 MySQL
5.7，[下载页面的地址](https://dev.mysql.com/downloads/mysql/5.7.html#downloads)，根据你们到系统安装对应的版本，Mac的话双击 dmg 安装包就可以按照提示一步步执行到安装成功。启动 MySQL，如下图：

![](https://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/images/56BgCn.jpg)

出现绿色就证明 MySQL 服务启动成功了。后面我们操作数据库不会通过本地命令行来，而是会通过图形化软件，比如：Navicat、Sequel
pro，这些图形化软件可比命令行的效率高太多，读者可以自行下载安装一下。

### Kafka 安装与配置

后面我们文章中会大量用到 Kafka，所以 Kakfa一定要安装好。官网下载地址：[下载页面的地址](https://kafka.apache.org/downloads)同样，我自己下载的版本是 1.1.0 （保持和我公司的生产环境一致），如果你对 Kafka 还不太熟悉，可以参考我以前写的一篇入门文章：[Kafka安装及快速入门](http://www.54tianzhisheng.cn/2018/01/04/Kafka/)。

在这篇文章里面教大家怎么安装 Kafka、启动 Zookeeper、启动 Kafka 服务、创建 Topic、使用 producer 创建消息、使用consumer 消费消息、查看 Topic 的信息，另外还有提供集群配置的方案。

### ElasticSearch 安装与配置

因为后面有文章介绍连接器 (connector) —— Elasticsearch 介绍和整和使用，并且最后面的案例文章也会把数据存储在Elasticsearch 中的，所以这里就简单的讲解一下 Elasticsearch 的安装，在我以前的博客中写过一篇搭建 Elasticsearch集群的：[Elasticsearch 系列文章（二）：全文搜索引擎 Elasticsearch
集群搭建入门教程](http://www.54tianzhisheng.cn/2017/09/09/Elasticsearch-install/)。

这里我在本地安装个单机的 Elasticsearch 就行了，首先在官网[下载页面](https://www.elastic.co/cn/downloads/past-releases) 找到 Elasticsearch产品，我下载的版本是 elasticsearch-6.3.2 版本，同样和我们公司的线上环境版本保持一致，因为 Flink Elasticsearchconnector 有分好几个版本：2.x、5.x、6.x 版本，不同版本到时候写数据存入到 Elasticsearch 的 Job代码也是有点区别的，如果你们公司的 Elasticsearch 版本比较低的话，到时候后面版本的学习代码还得找官网的资料对比学习一下。

另外就是写这篇文章的时候 Elasticsearch 7.x 就早已经发布了，Flink 我暂时还没看到支持 Elasticsearch 7
的连接器，自己也没测试过，所以暂不清楚如果用 6.x 版本的 connector 去连接 7.x 的 Elasticsearch
会不会出现问题？建议还是跟着我的安装版本来操作！

除了这样下载 Elasticsearch 的话，你如果电脑安装了 Homebrew，也可以通过 Homebrew 来安装
Elasticsearch，都还挺方便的，包括你还可以通过 Docker 的方式快速启动一个 Elasticsearch 来。

下载好了 Elasticsearch 的压缩包，在你的安装目录下解压就行了，然后进入 Elasticsearch 的安装目录执行下面命令就可以启动
Elasticsearch 了：


​    
    ./bin/elasticsearch


执行命令后的结果：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-17-144155.png)

从浏览器端打开地址：`http://localhost:9200/` 即可验证是否安装成功：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-04-25-003-es.png)

如果出现了如上图这样就代表 Elasticsearch 环境已经安装好了。

### 小结与反思

本节讲解了下 JDK、Maven、IDE、MySQL、Kafka、ElasticSearch
的安装与配置，因为这些都是后面要用的，所以这里单独抽一篇文章来讲解环境准备的安装步骤，当然这里还并不涉及全，因为后面我们还可能会涉及到 HBase、HDFS等知识，后面我们用到再看，我们本系列的文章更多的还是讲解 Flink，所以更多的环境准备还是得靠大家自己独立完成。

这里我说下笔者自己一般安装环境的选择：

  1. 组件尽量和公司的生产环境保持版本一致，不追求太新，够用就行，这样如果生产出现问题，本机还可以看是否可以复现出来

  2. 安装环境的时候先搜下类似的安装教程，提前知道要踩的坑，避免自己再次踩到

下面文章我们就正式进入 Flink 专题了！

