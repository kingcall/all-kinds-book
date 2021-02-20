大数据领域SQL化开发的风潮方兴未艾（所谓"Everybody knows SQL"），Flink自然也不能“免俗”。Flink SQL是Flink系统内部最高级别的API，也是流批一体思想的集大成者。用户可以通过简单明了的SQL语句像查表一样执行流任务或批任务，屏蔽了底层DataStream/DataSet API的复杂细节，降低了使用门槛。

但是，Flink SQL的默认开发方式是通过Java/Scala API编写，与纯SQL化、平台化的目标相去甚远。目前官方提供的Flink SQL Client仅能在配备Flink客户端的本地使用，局限性很大。而Ververica开源的Flink SQL Gateway组件是基于REST API的，仍然需要二次开发才能供给上层使用，并不是很方便。

鉴于有很多企业都无法配备专门的团队来解决Flink SQL平台化的问题，那么到底有没有一个开源的、开箱即用的、功能相对完善的组件呢？答案就是本文的主角——Apache Zeppelin。

### Flink SQL on Zeppelin！

![图片](https://mmbiz.qpic.cn/mmbiz_png/sq2uE6cicHYxnk4UQicNNIVo8AjYFcKic80OzkKBW31VHoBxz5F3Xw5k2iapzkbtZu3qmklqfRQYOkt0MubIHbX9AA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

Zeppelin是基于Web的交互式数据分析笔记本，支持SQL、Scala、Python等语言。Zeppelin通过插件化的Interpreter（解释器）来解析用户提交的代码，并将其转化到对应的后端（计算框架、数据库等）执行，灵活性很高。其架构简图如下所示。

![图片](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/640-20210219214933363.png)

Flink Interpreter就是Zeppelin原生支持的众多Interpreters之一。只要配置好Flink Interpreter以及相关的执行环境，我们就可以将Zeppelin用作Flink SQL作业的开发平台了（当然，Scala和Python也是没问题的）。接下来本文就逐步介绍Flink on Zeppelin的集成方法。

### 配置Zeppelin

目前Zeppelin的最新版本是0.9.0-preview2，可以在官网下载包含所有Interpreters的zeppelin-0.9.0-preview2-bin-all.tgz，并解压到服务器的合适位置。

接下来进入conf目录。将环境配置文件zeppelin-env.sh.template更名为zeppelin-env.sh，并修改：

```
# JDK目录
export JAVA_HOME=/opt/jdk1.8.0_172
# 方便之后配置Interpreter on YARN模式。注意必须安装Hadoop，且hadoop必须配置在系统环境变量PATH中
export USE_HADOOP=true
# Hadoop配置文件目录
export HADOOP_CONF_DIR=/etc/hadoop/hadoop-conf
```

将服务配置文件zeppelin-site.xml.template更名为zeppelin-site.xml，并修改：

```
<!-- 服务地址。默认为127.0.0.1，改为0.0.0.0使得可以在外部访问 -->
<property>
  <name>zeppelin.server.addr</name>
  <value>0.0.0.0</value>
  <description>Server binding address</description>
</property>

<!-- 服务端口。默认为8080，如果已占用，可以修改之 -->
<property>
  <name>zeppelin.server.port</name>
  <value>18080</value>
  <description>Server port.</description>
</property>
```

最基础的配置就完成了。运行bin/zeppelin-daemon.sh start命令，返回Zeppelin start [ OK ]的提示之后，访问<服务器地址>:18080，出现下面的页面，就表示Zeppelin服务启动成功。

![图片](https://mmbiz.qpic.cn/mmbiz_png/sq2uE6cicHYxnk4UQicNNIVo8AjYFcKic80zlzhqqLbCzHdtbL10iadiahX6cdlkNMhUiaj2mUB0952reBbIsSFzSMLQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

当然，为了一步到位适应生产环境，也可以适当修改zeppelin-site.xml中的以下参数：

```
<!-- 将Notebook repo更改为HDFS存储 -->
<property>
  <name>zeppelin.notebook.storage</name>
  <value>org.apache.zeppelin.notebook.repo.FileSystemNotebookRepo</value>
  <description>Hadoop compatible file system notebook persistence layer implementation, such as local file system, hdfs, azure wasb, s3 and etc.</description>
</property>

<!-- Notebook在HDFS上的存储路径 -->
<property>
  <name>zeppelin.notebook.dir</name>
  <value>/zeppelin/notebook</value>
  <description>path or URI for notebook persist</description>
</property>

<!-- 启用Zeppelin的恢复功能。当Zeppelin服务挂掉并重启之后，能连接到原来运行的Interpreter -->
<property>
  <name>zeppelin.recovery.storage.class</name>
  <value>org.apache.zeppelin.interpreter.recovery.FileSystemRecoveryStorage</value>
  <description>ReoveryStorage implementation based on hadoop FileSystem</description>
</property>

<!-- Zeppelin恢复元数据在HDFS上的存储路径 -->
<property>
  <name>zeppelin.recovery.dir</name>
  <value>/zeppelin/recovery</value>
  <description>Location where recovery metadata is stored</description>
</property>

<!-- 禁止使用匿名用户 -->
<property>
  <name>zeppelin.anonymous.allowed</name>
  <value>true</value>
  <description>Anonymous user allowed by default</description>
</property>
```

Zeppelin集成了Shiro实现权限管理。禁止使用匿名用户之后，可以在conf目录下的shiro.ini中配置用户名、密码、角色等，不再赘述。注意每次修改配置都需要运行bin/zeppelin-daemon.sh restart重启Zeppelin服务。

### 配置Flink Interpreter on YARN

在使用Flink Interpreter之前，我们有必要对它进行配置，使Flink作业和Interpreter本身在YARN环境中运行。

点击首页用户名区域菜单中的Interpreter项（上一节图中已经示出），搜索flink，就可以看到参数列表。

![图片](https://mmbiz.qpic.cn/mmbiz_png/sq2uE6cicHYxnk4UQicNNIVo8AjYFcKic80NicutkibpeEXjfolibic4hZeBnEvIYoiap5QNv3gAjycWHiabTzsOibfxnvNQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### Interpreter Binding

首先，将Interpreter Binding模式修改为Isolated per Note，如下图所示。

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在这种模式下，每个Note在执行时会分别启动Interpreter进程，类似于Flink on YARN的Per-job模式，最符合生产环境的需要。

### Flink on YARN参数

以下是需要修改的部分基础参数。注意这些参数也可以在Note中指定，每个作业自己的配置会覆盖掉这里的默认配置。

- FLINK_HOME：Flink 1.11所在的目录；
- HADOOP_CONF_DIR：Hadoop配置文件所在的目录；
- flink.execution.mode：Flink作业的执行模式，指定为yarn以启用Flink on YARN；
- flink.jm.memory：JobManager的内存量（MB）；
- flink.tm.memory：TaskManager的内存量（MB）；
- flink.tm.slot：TaskManager的Slot数；
- flink.yarn.appName：YARN Application的默认名称；
- flink.yarn.queue：提交作业的默认YARN队列。

#### Hive Integration参数

如果我们想访问Hive数据，以及用HiveCatalog管理Flink SQL的元数据，还需要配置与Hive的集成。

- HIVE_CONF_DIR：Hive配置文件（hive-site.xml）所在的目录；

- zeppelin.flink.enableHive：设为true以启用Hive Integration；

- zeppelin.flink.hive.version：Hive版本号。

- 复制与Hive Integration相关的依赖到$FLINK_HOME/lib目录下，包括：

  ```
    flink-connector-hive_2.11-1.11.0.jar
    flink-hadoop-compatibility_2.11-1.11.0.jar
    hive-exec-*.*.jar
    如果Hive版本是1.x，还需要额外加入hive-metastore-1.*.jar、libfb303-0.9.2.jar和libthrift-0.9.2.jar
  ```

- 保证Hive元数据服务（Metastore）启动。注意不能是Embedded模式，即必须以外部数据库（MySQL、Postgres等）作为元数据存储。

#### Interpreter on YARN参数

在默认情况下，Interpreter进程是在部署Zeppelin服务的节点上启动的。随着提交的任务越来越多，就会出现单点问题。因此我们需要让Interpreter也在YARN上运行，如下图所示。

![图片](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/640-20210219214934089.png)

- zeppelin.interpreter.yarn.resource.cores：Interpreter Container占用的vCore数量
- zeppelin.interpreter.yarn.resource.memory：Interpreter Container占用的内存量（MB）
- zeppelin.interpreter.yarn.queue：Interpreter所处的YARN队列名称

配置完成之后，Flink on Zeppelin集成完毕，可以测试一下了。

### 测试Flink SQL on Zeppelin

创建一个Note，Interpreter指定为flink。然后写入第一个Paragraph：

![图片](https://mmbiz.qpic.cn/mmbiz_png/sq2uE6cicHYxnk4UQicNNIVo8AjYFcKic805RExRMdicRBHVJIEEh8ibFaibB40vftCF0ecnvibCicgkXjssMl41eTS0dg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

以%flink.conf标记的Paragraph用于指定这个Note中的作业配置，支持Flink的所有配置参数（参见Flink官网）。另外，flink.execution.packages参数支持以Maven GAV坐标的方式引入外部依赖项。

接下来创建第二个Paragraph，创建Kafka流表：

![图片](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/640-20210219214934686.png)

%flink.ssql表示利用StreamTableEnvironment执行流处理SQL，相对地，%flink.bsql表示利用BatchTableEnvironment执行批处理SQL。注意表参数中的properties.bootstrap.servers利用了Zeppelin Credentials来填写，方便不同作业之间复用。

执行上述SQL之后会输出信息：

![图片](https://mmbiz.qpic.cn/mmbiz_png/sq2uE6cicHYxnk4UQicNNIVo8AjYFcKic80eiaNPD3LlugiaDjAuK4vF6Rvsb9hg5cZJ9Cic7iayoSJeGaOvR7maGX2nA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

同时在Hive中可以看到该表的元数据。最后写第三个Paragraph，从流表中查询，并实时展现出来：

![image-20210219215019597](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210219215019597.png)

点击右上角的FLINK JOB标记，可以打开作业的Web UI。上述作业的JobGraph如下。

![image-20210219215004443](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/image-20210219215004443.png)

除SELECT查询外，通过Zeppelin也可以执行INSERT查询，实现更加丰富的功能。关于Flink SQL on Zeppelin的更多应用，笔者在今后的文章中会继续讲解，今天就到这里吧。