# 使用Intellij idea编写Spark应用程序(Scala+SBT)

对Scala代码进行打包编译时，可以采用Maven，也可以采用SBT，相对而言，业界更多使用SBT。

## 运行环境

Ubuntu 16.04
Spark 2.1.0
Intellij Idea (Version 2017.1)

## 安装Scala插件

安装Scala插件,该Scala插件自带SBT工具。如果已经安装Scala插件，即可跳过此步骤
![intellij idea 安装scala插件](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570548971000_20191008233614265164-20210112091220906.png)

![安装scala插件](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570634405000_20191009232014018637-20210112091229189.png)

![Intellij idea安装scala插件](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570634465000_20191009232109815021-20210112091233872.png)
点击Install,即可安装Scala插件。

## 构建基于SBT的Scala项目

如下图，按顺序执行如下操作:
新建项目
![新建项目](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570634513000_20191009232157832972-20210112091239031.png)

选择Scala—>SBT
![选择sbt](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570634541000_20191009232226525369-20210112091224109-20210112091242867.png)

设置项目名，点击Finish即可。
![设置项目名称](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570634564000_20191009232247408598-20210112091247432.png)
这里需要设置Scala的版本必须2.11.*的版本号。因为Spark 2.0是基于Scala 2.11构建的。这个可以在Spark的官网查到，如下图：
![scala版本](http://www.hadoopdoc.com/media/editor/file_1570634600000_20191009232323534007.png)

## 利用SBT 添加依赖包

利用Spark的官网查到Spark artifacts的相关版本号，如下图：
![spark artifacts 版本号](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570634626000_20191009232351088370-20210112091254157.png)

编辑Intellij Idea项目中是build.sbt:

```
name := "SBTTest"version := "1.0"scalaVersion := "2.11.8"libraryDependencies += "org.apache.spark" %% "spark-core" % "2.1.0"
```

编辑后，Intellij Idea弹出提示，如图：
![intellij 弹出窗口](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570634671000_20191009232433586800-20210112091259991.png)

可以选择Refresh Project手动刷新，也可以选择Enable auto-import让Intellij Idea以后每次遇到build.sbt更新后自动导入依赖包。
这里，选择Enable auto-import.

## 创建WordCount实例

在Linux系统中新建一个命令行终端（Shell环境），在终端中执行如下命令,新建word.txt测试文件:

```
echo "hadoop hello spark hello world" >> ~/word.txt
```

在Intellij Idea的src/main/scala项目目录下新建WordCount.scala文件，如下图（注意看图下面的备注）：

![新建WordCount.scala文件](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570634721000_20191009232525915873-20210112091307104.png)

**备注**：这里需要注意，在Intellij Idea启动时，会执行“dump project structure from sbt”的操作，也就是把sbt所需要的项目结构从远程服务器拉取到本地，在本地会生成sbt所需要的项目结构。由于是从国外的远程服务器下载，所以，这个过程很慢。这个过程没有结束之前，上图中的“File->New”弹出的子菜单是找不到Scala Class这个选项的。所以，一定要等“dump project structure from sbt”的操作全部执行结束以后，再去按照上图操作来新建Scala Class文件。备注：这里需要注意，在Intellij Idea启动时，会执行“dump project structure from sbt”的操作，也就是把sbt所需要的项目结构从远程服务器拉取到本地，在本地会生成sbt所需要的项目结构。由于是从国外的远程服务器下载，所以，这个过程很慢。这个过程没有结束之前，上图中的“File->New”弹出的子菜单是找不到Scala Class这个选项的。所以，一定要等“dump project structure from sbt”的操作全部执行结束以后，再去按照上图操作来新建Scala Class文件。

新建Scala Class文件的代码如下：

```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf
import org.apache.log4j.{Level,Logger}
object WordCount {
  def main(args: Array[String]) {
    //屏蔽日志
    Logger.getLogger("org.apache.spark").setLevel(Level.WARN)
    Logger.getLogger("org.eclipse.jetty.server").setLevel(Level.OFF)
    val inputFile =  "file:///home/hadoop/word.txt"
    val conf = new SparkConf().setAppName("WordCount").setMaster("local[2]")
    val sc = new SparkContext(conf)
    val textFile = sc.textFile(inputFile)
    val wordCount = textFile.flatMap(line => line.split(" ")).map(word => (word, 1)).reduceByKey((a, b) => a + b)
    wordCount.foreach(println)
  }
}

```

右键WordCount.scala,选择执行该文件，如下图：
![执行WordCount.scala](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570634854000_20191009232737905950-20210112091315262.png)

即可在Intellij Idea下面看到输出结果。