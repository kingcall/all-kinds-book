## Flink 环境搭建

[TOC]

在 2.1 节中已经将 Flink 的准备环境已经讲完了，本篇文章将带大家正式开始接触 Flink，那么我们得先安装一下 Flink。Flink
是可以在多个平台（Windows、Linux、Mac）上安装的。在开始写本书的时候最新版本是 1.8 版本，但是写到一半后更新到 1.9 了（合并了大量
Blink 的新特性），所以笔者又全部更新版本到 1.9，书籍后面也都是基于最新的版本讲解与演示。

Flink 的官网地址是：<https://flink.apache.org/>

### Flink 下载与安装

#### Mac & Linux 安装

你可以通过该地址 <https://flink.apache.org/downloads.html> 下载到最新版本的 Flink。

这里我们选择 `Apache Flink 1.9.0 for Scala 2.11`
版本，点击跳转到了一个镜像下载选择的地址，随便选择哪个就行，只是下载速度不一致而已。

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-04-25-001-flink-mimor.png)

下载完后，你就可以直接解压下载的 Flink 压缩包了。

接下来我们可以启动一下 Flink，我们进入到 Flink 的安装目录下执行命令 `./bin/start-cluster.sh` 即可，产生的日志如下：


​    
    zhisheng@zhisheng /usr/local/flink-1.9.0  ./bin/start-cluster.sh
    Starting cluster.
    Starting standalonesession daemon on host zhisheng.
    Starting taskexecutor daemon on host zhisheng.


如果你的电脑是 Mac 的话，那么你也可以通过 Homebrew 命令进行安装。先通过命令 `brew search flink` 查找一下包：

     zhisheng@zhisheng  ~  brew search flink
    ==> Formulae
    apache-flink ✔       homebrew/linuxbrew-core/apache-flink


可以发现找得到 Flink 的安装包，但是这样安装的版本可能不是最新的，如果你要安装的话，则使用命令：

    brew install apache-flink


那么它就会开始进行下载并安装好，安装后的目录应该是在 `/usr/local/Cellar/apache-flink` 下。

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-23-030606.png)

你可以通过下面命令检查安装的 Flink 到底是什么版本的：

    flink --version


结果：

    Version: 1.9.0, Commit ID: ff472b4


这种的话运行是得进入 `/usr/local/Cellar/apache-flink/1.9.0/libexec/bin` 目录下执行命令
`./start-cluster.sh` 才可以启动 Flink 的。

启动后产生的日志：

    Starting cluster.
    Starting standalonesession daemon on host zhisheng.
    Starting taskexecutor daemon on host zhisheng.


#### Windows 安装

如果你的电脑系统是 Windows 的话，那么你就直接双击 Flink 安装目录下面 bin 文件夹里面的 `start-cluster.bat`
就行，同样可以将 Flink 起动成功。

### Flink 启动与运行

启动成功后的话，我们可以通过访问地址`http://localhost:8081/` 查看 UI 长啥样了，如下图所示：

![](http://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/img/2019-10-23-021138.png)

你在通过 jps 命令可以查看到运行的进程有：


​    
    zhisheng@zhisheng  /usr/local/flink-1.9.0  jps
    73937 StandaloneSessionClusterEntrypoint
    74391 Jps
    520
    74362 TaskManagerRunner

### Flink 目录配置文件解读

Flink 安装好后，我们也运行启动看了效果了，接下来我们来看下它的目录结构吧：

     ✘ zhisheng@zhisheng  /usr/local/flink-1.9.0  ll
    total 1200
    -rw-r--r--@  1 zhisheng  staff    11K  3  5 16:32 LICENSE
    -rw-r--r--@  1 zhisheng  staff   582K  4  4 00:01 NOTICE
    -rw-r--r--@  1 zhisheng  staff   1.3K  3  5 16:32 README.txt
    drwxr-xr-x@ 26 zhisheng  staff   832B  3  5 16:32 bin
    drwxr-xr-x@ 14 zhisheng  staff   448B  4  4 14:06 conf
    drwxr-xr-x@  6 zhisheng  staff   192B  4  4 14:06 examples
    drwxr-xr-x@  5 zhisheng  staff   160B  4  4 14:06 lib
    drwxr-xr-x@ 47 zhisheng  staff   1.5K  3  6 23:21 licenses
    drwxr-xr-x@  2 zhisheng  staff    64B  3  5 19:50 log
    drwxr-xr-x@ 22 zhisheng  staff   704B  4  4 14:06 opt


上面目录：

  * **bin** 存放一些启动脚本
  * **conf** 存放配置文件
  * **examples** 存放一些案例的 Job Jar 包
  * **lib** Flink 依赖的 Jar 包
  * **log** 存放产生的日志文件
  * **opt** 存放的是一些可选择的 Jar 包，后面可能会用到

在 bin 目录里面有如下这些脚本：


​    
    zhisheng@zhisheng  /usr/local/flink-1.9.0  ll bin
    total 256
    -rwxr-xr-x@ 1 zhisheng  staff    28K  3  5 16:32 config.sh
    -rwxr-xr-x@ 1 zhisheng  staff   2.2K  3  5 16:32 flink
    -rwxr-xr-x@ 1 zhisheng  staff   2.7K  3  5 16:32 flink-console.sh
    -rwxr-xr-x@ 1 zhisheng  staff   6.2K  3  5 16:32 flink-daemon.sh
    -rwxr-xr-x@ 1 zhisheng  staff   1.2K  3  5 16:32 flink.bat
    -rwxr-xr-x@ 1 zhisheng  staff   1.5K  3  5 16:32 historyserver.sh
    -rwxr-xr-x@ 1 zhisheng  staff   2.8K  3  5 16:32 jobmanager.sh
    -rwxr-xr-x@ 1 zhisheng  staff   1.8K  3  5 16:32 mesos-appmaster-job.sh
    -rwxr-xr-x@ 1 zhisheng  staff   1.8K  3  5 16:32 mesos-appmaster.sh
    -rwxr-xr-x@ 1 zhisheng  staff   1.8K  3  5 16:32 mesos-taskmanager.sh
    -rwxr-xr-x@ 1 zhisheng  staff   1.2K  3  5 16:32 pyflink-stream.sh
    -rwxr-xr-x@ 1 zhisheng  staff   1.1K  3  5 16:32 pyflink.bat
    -rwxr-xr-x@ 1 zhisheng  staff   1.1K  3  5 16:32 pyflink.sh
    -rwxr-xr-x@ 1 zhisheng  staff   3.4K  3  5 16:32 sql-client.sh
    -rwxr-xr-x@ 1 zhisheng  staff   2.5K  3  5 16:32 standalone-job.sh
    -rwxr-xr-x@ 1 zhisheng  staff   3.3K  3  5 16:32 start-cluster.bat
    -rwxr-xr-x@ 1 zhisheng  staff   1.8K  3  5 16:32 start-cluster.sh
    -rwxr-xr-x@ 1 zhisheng  staff   3.3K  3  5 16:32 start-scala-shell.sh
    -rwxr-xr-x@ 1 zhisheng  staff   1.8K  3  5 16:32 start-zookeeper-quorum.sh
    -rwxr-xr-x@ 1 zhisheng  staff   1.6K  3  5 16:32 stop-cluster.sh
    -rwxr-xr-x@ 1 zhisheng  staff   1.8K  3  5 16:32 stop-zookeeper-quorum.sh
    -rwxr-xr-x@ 1 zhisheng  staff   3.8K  3  5 16:32 taskmanager.sh
    -rwxr-xr-x@ 1 zhisheng  staff   1.6K  3  5 16:32 yarn-session.sh
    -rwxr-xr-x@ 1 zhisheng  staff   2.2K  3  5 16:32 zookeeper.sh


脚本包括了配置启动脚本、historyserver、Job Manager、Task Manager、启动集群和停止集群等脚本。

在 conf 目录下面有如下这些配置文件：

    zhisheng@zhisheng  /usr/local/flink-1.9.0  ll conf
    total 112
    -rw-r--r--@ 1 zhisheng  staff   9.8K  4  4 00:01 flink-conf.yaml
    -rw-r--r--@ 1 zhisheng  staff   2.1K  3  5 16:32 log4j-cli.properties
    -rw-r--r--@ 1 zhisheng  staff   1.8K  3  5 16:32 log4j-console.properties
    -rw-r--r--@ 1 zhisheng  staff   1.7K  3  5 16:32 log4j-yarn-session.properties
    -rw-r--r--@ 1 zhisheng  staff   1.9K  3  5 16:32 log4j.properties
    -rw-r--r--@ 1 zhisheng  staff   2.2K  3  5 16:32 logback-console.xml
    -rw-r--r--@ 1 zhisheng  staff   1.5K  3  5 16:32 logback-yarn.xml
    -rw-r--r--@ 1 zhisheng  staff   2.3K  3  5 16:32 logback.xml
    -rw-r--r--@ 1 zhisheng  staff    15B  3  5 16:32 masters
    -rw-r--r--@ 1 zhisheng  staff    10B  3  5 16:32 slaves
    -rw-r--r--@ 1 zhisheng  staff   3.8K  3  5 16:32 sql-client-defaults.yaml
    -rw-r--r--@ 1 zhisheng  staff   1.4K  3  5 16:32 zoo.cfg


配置包含了 Flink 的自身配置、日志配置、masters、slaves、sql-client、zoo 等配置。

在 examples 目录里面可以看到有如下这些案例的目录：


​    
    zhisheng@zhisheng  /usr/local/flink-1.9.0  ll examples
    total 0
    drwxr-xr-x@ 10 zhisheng  staff   320B  4  4 14:06 batch
    drwxr-xr-x@  3 zhisheng  staff    96B  4  4 14:06 gelly
    drwxr-xr-x@  4 zhisheng  staff   128B  4  4 14:06 python
    drwxr-xr-x@ 11 zhisheng  staff   352B  4  4 14:06 streaming


这个目录下面有批、gelly、python、流的 demo，后面我们可以直接用上面的案例做些简单的测试。

在 log 目录里面存着 Task Manager & Job manager 的日志：

    zhisheng@zhisheng  /usr/local/flink-1.9.0  ll log
    total 144
    -rw-r--r--  1 zhisheng  staff    11K  4 25 20:10 flink-zhisheng-standalonesession-0-zhisheng.log
    -rw-r--r--  1 zhisheng  staff     0B  4 25 20:10 flink-zhisheng-standalonesession-0-zhisheng.out
    -rw-r--r--  1 zhisheng  staff    11K  4 25 20:10 flink-zhisheng-taskexecutor-0-zhisheng.log
    -rw-r--r--  1 zhisheng  staff     0B  4 25 20:10 flink-zhisheng-taskexecutor-0-zhisheng.out


一般我们如果要深入了解一个知识点，最根本的方法就是看其源码实现，源码下面无秘密，所以我这里也讲一下如何将源码下载编译并运行，然后将代码工程导入到 IDEA中去，方便自己查阅和 debug 代码。

### Flink 源码下载

Flink GitHub 仓库地址：<https://github.com/apache/flink>

执行下面命令将源码下载到本地：    

    git clone git@github.com:apache/flink.git


拉取的时候找个网络好点的地方，这样速度可能会更快点。

然后你可以切换到项目的不同分支，比如 release-1.9、blink（阿里巴巴开源贡献的） ，执行下面命令将代码切换到 release-1.9 分支：

    git checkout release-1.9


或者你也想去看看 Blink 的代码实现，你也可以执行下面命令切换到 blink 分支来    

    git checkout blink

### Flink 源码编译

编译源码的话，你需要执行如下命令：

    mvn clean install -Dmaven.test.skip=true -Dmaven.javadoc.skip=true -Dcheckstyle.skip=true


  * -Dmaven.test.skip：跳过测试代码
  * -Dmaven.javadoc.skip：跳过 javadoc 检查
  * -Dcheckstyle.skip：跳过代码风格检查

maven 编译的时候跳过这些检查，这样可以减少很多时间，还可能会减少错误的发生。

注意：你的 maven 的 settings.xml 文件的 mirror 添加下面这个(这样才能下载到某些下载不了的依赖)。

    <mirror>
      <id>nexus-aliyun</id>
      <mirrorOf>*,!jeecg,!jeecg-snapshots,!mapr-releases</mirrorOf>
      <name>Nexus aliyun</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
    
    <mirror>
      <id>mapr-public</id>
      <mirrorOf>mapr-releases</mirrorOf>
      <name>mapr-releases</name>
      <url>https://maven.aliyun.com/repository/mapr-public</url>
    </mirror>


如果还遇到什么其他的问题的话，可以去看看我之前在我博客分享的一篇源码编译的文章（附视频）：[Flink 源码解析 ——源码编译运行](http://www.54tianzhisheng.cn/2019/01/30/Flink-code-compile/)。

### Flink 源码导入到 IDE

看下图，因为我们已经下载好了源码，直接在 IDEA 里面 open 这个 maven 项目就行了：

![](https://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/images/1CP82X.jpg)

导入后大概就是下面这样子：

![](https://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/images/olmtP4.jpg)

很顺利，没多少报错，这里我已经把一些代码风格检查相关的 Maven 插件给注释掉了。

### 小结与反思

本节主要讲了 FLink 在不同系统下的安装和运行方法，然后讲了下怎么去下载源码和将源码导入到 IDE 中。不知道你在将源码导入到 IDE
中是否有遇到什么问题呢？

