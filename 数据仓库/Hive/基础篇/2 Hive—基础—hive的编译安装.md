[TOC]

## 编译

作为一个大数据开发工程师，基本大大数组件的编译安装你的会，至少你得尝试编译，因为很多时候你都会遇到版本冲突的问题，这个时候你就得去选择合适的依赖版本，然后去重新打包，或者是需要做自定义开发，这个时候你还是需要自己去编译安装

所以这里我就不建议你直接去下载已经编译好的安装包了，但是我得告诉你有这个东西，例如你可以去清华的镜像网站`https://mirrors.tuna.tsinghua.edu.cn/apache/hive/` 去查看hive 的二进制安装包

![image-20201222212158667](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/22/21:21:59-image-20201222212158667.png)

### clone 源码

git clone https://github.com/apache/hive.git

在clone 成功之后，你可以手动切换特定的版本分支，也可以将其导入idea 然后切换分支，这里我就导入idea 了，因为这样方便你阅读源码

在idea 打开之后，我选择了一个特定的分支，也就是branc-3.1 默认是在master 分支的

![image-20201222201703371](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/22/20:49:00-20:17:03-image-20201222201703371.png)

### 查看你的hadoop 版本

因为hive 是依赖hadoop ，所以编译之前你需要知道你的hadoop 版本，你可以使用`hadoop version` 进行查看

![image-20201222201018972](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/22/20:10:20-image-20201222201018972.png)



### 编译

在开始之前编译之前我们先说点其他的，你如果去网上搜的话，很多人说让你使用`mvn clean package -Phadoop-2 -DskipTests ` 这样的命令打包，这个时候你得看一下是否有这样的profile 

否则当你打包结束之后，会有一个警告说是不存在这hadoop-2 这样的profile 

这里我们打开hive 的pom 文件，发现里面hadoop 的版本是通过hadoop.version 这个properties 定义的

![image-20201222201601334](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/22/20:16:01-image-20201222201601334.png)

这个时候我们就可以使用下面的命令进行打包我们想要的版本` mvn clean package -Dhadoop.version=3.2.1 -DskipTests -Dmaven.javadoc.skip=true`

![image-20201222212755933](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/22/21:27:57-image-20201222212755933.png)

你就会发现这样打包的话，完成之后连个警告都没有

如果你打包过程中出现了jar 找不到的话，请使用maven 官方的镜像，或者是配置多个镜像，至少得让在自定镜像找不到的时候可以去官方镜像进行查找

## 安装

### 解压到安装位置

打包结束之后，去`packaging/target` 目录下就可以找到打好的安装包

![image-20201222213123235](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/22/21:31:23-image-20201222213123235.png)

接下来就是将 apache-hive-3.1.3-bin.tar.gz 解压到特定位置

### Hadoop 环境准备

这里我就认为你hadoop 环境已经安装好了

这里我们需要创建两个文件夹，/user/hive/warehouse 就是将来数仓的位置( `hive.metastore.warehouse.dir`)

```
  $ $HADOOP_HOME/bin/hadoop fs -mkdir       /tmp
  $ $HADOOP_HOME/bin/hadoop fs -mkdir       /user/hive/warehouse
  $ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /tmp
  $ $HADOOP_HOME/bin/hadoop fs -chmod g+w   /user/hive/warehouse
  
```

### 修改hive-site.xml

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?><?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
        <property>
            <name>javax.jdo.option.ConnectionURL</name>
            <value>jdbc:mysql://localhost/hive</value>
        </property>
        <property>
           <name>javax.jdo.option.ConnectionDriverName</name>
           <value>com.mysql.jdbc.Driver</value>
        </property>
        <property>
        		<name>javax.jdo.option.ConnectionUserName</name>
            <value>root</value>
        </property>
        <property>
            <name>javax.jdo.option.ConnectionPassword</name>
            <value>www1234</value>
        </property>
        <property>
            <name>hive.metastore.schema.verification</name>
           	<value>false</value>
        </property>
        <property> 
            <name>hive.cli.print.current.db</name>
            <value>true</value>
        </property>
        <property> 
            <name>hive.cli.print.header</name>
            <value>true</value>
        </property>
	<!-- 这是hiveserver2 -->
      <property>
               <name>hive.server2.thrift.port</name>
             <value>10000</value>
      </property>
    	<property>
       		<name>hive.server2.thrift.bind.host</name>
       		<value>localhost</value>
     	</property>
</configuration>
```

关于其他配置还有很多，这里我们只需要最小配置让hive 成功运行即可，其实hiveserver2 可以不配置的，但是我们后面马上要用，所以我们就配置了

### 初始化元数据库

```
 $HIVE_HOME/bin/schematool -dbType mysql -initSchema
```

因为我这里是mysql 所以我-dbType 参数对于的就是mysql ，其实hive是支持三种安装模式的

1、内嵌模式（元数据保村在内嵌的derby种，允许一个会话链接，尝试多个会话链接时会报错，不适合开发环境）
2、本地模式（本地安装mysql 替代derby存储元数据）
3、远程模式（远程安装mysql 替代derby存储元数据）

当然如果你在初始化的过程中遇到了什么问题的话，你可以使用下面的命令来查看详细信息

```
schematool -dbType mysql -initSchema  --verbose 
```

例如

```
Loading class `com.mysql.jdbc.Driver'. This is deprecated. The new driver class is `com.mysql.cj.jdbc.Driver'. The driver is automatically registered via the SPI and manual loading of the driver class is generally unnecessary.
```

解决方法：打开 hive_site.xml文件，把`com.mysql.jdbc.Driver改成com.mysql.cj.jdbc.Driver`

只要你能拿到报错日志，就可以在网上去找响应的解决方案了

### 启动metastore

```
 nohup hive --service metastore &
```

到这里其实我们已经完成了整个Hive 的安装和最小配置,接下来我们看一下服务是否启动成功了

![image-20201222220353850](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/22/22:03:54-image-20201222220353850.png)

### 愉快的使用吧

其实我已使用了，建了一个ods 数据库，哈哈，日志有点多啊

![image-20201222220532333](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/22/22:05:32-image-20201222220532333.png)