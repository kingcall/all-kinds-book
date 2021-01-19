# Spark GraphX 算法实例

GraphX 中自带一系列图算法来简化分析任务。这些算法存在于org.apache.spark.graphx.lib包中，可以被Graph通过GraphOps直接访问。本章节主要介绍GraphX中主要的三个算法。

## PageRank算法

PageRank，有成网页排名算法。PageRank通过网络的超链接关系确定网页的等级好坏，在搜索引擎优化操作中常用来评估网页的相关性和重要性。

PageRank同样可以在图中测量每个顶点的重要性，假设存在一条从顶点u到顶点v的边，就代表顶点u对顶点v的支持。例如：微博中，一个用户被其他很多用户关注，那么这个用户的排名将会很高。

GraphX 自带静态和动态的PageRank算法实现。静态的PageRank算法运行在固定的迭代次数，动态的PageRank算法运行直到整个排名收敛(eg:通过限定可容忍的值来停止迭代)。

利用GraphX自带的社会网络数据集实例，用户集合数据集存在/usr/local/Spark/data/graphx/users.txt，用户关系数据集存在/usr/local/Spark/data/graphx/followers.txt。现在计算每个用户的PageRank：

```
import org.apache.log4j.{Level,Logger}
import org.apache.spark._
import org.apache.spark.graphx.GraphLoader
object SimpleGraphX {
  def main(args: Array[String]) {
    //屏蔽日志
    Logger.getLogger("org.apache.spark").setLevel(Level.WARN)
    Logger.getLogger("org.eclipse.jetty.server").setLevel(Level.OFF)
    //设置运行环境
    val conf = new SparkConf().setAppName("SimpleGraphX").setMaster("local")
    val sc = new SparkContext(conf)
    // Load the edges as a graph
    val graph = GraphLoader.edgeListFile(sc, "file:///usr/local/Spark/data/graphx/followers.txt")
    // Run PageRank
    val ranks = graph.pageRank(0.0001).vertices
    // Join the ranks with the usernames
    val users = sc.textFile("file:///usr/local/Spark/data/graphx/users.txt").map { line =>
      val fields = line.split(",")
      (fields(0).toLong, fields(1))
    }
    val ranksByUsername = users.join(ranks).map {
      case (id, (username, rank)) => (username, rank)
    }
    // Print the result
    println(ranksByUsername.collect().mkString("\n"))
  }
}
```

## 连通分支算法

连通分支算法使用最小编号的顶点来标记每个连通分支。

在一个社会网络，连通图近似簇。这里我们计算一个连通分支实例，所使用的数据集和PageRank一样。

```
import org.apache.log4j.{Level,Logger}
import org.apache.spark._
import org.apache.spark.graphx.GraphLoader
object SimpleGraphX {
  def main(args: Array[String]) {
    //屏蔽日志
    Logger.getLogger("org.apache.spark").setLevel(Level.WARN)
    Logger.getLogger("org.eclipse.jetty.server").setLevel(Level.OFF)
    //设置运行环境
    val conf = new SparkConf().setAppName("SimpleGraphX").setMaster("local")
    val sc = new SparkContext(conf)
    // Load the edges as a graph
    // Load the graph as in the PageRank example
    val graph = GraphLoader.edgeListFile(sc, "file:///usr/local/Spark/data/graphx/followers.txt")
    // Find the connected components
    val cc = graph.connectedComponents().vertices
    // Join the connected components with the usernames
    val users = sc.textFile("file:///usr/local/Spark/data/graphx/users.txt").map { line =>
      val fields = line.split(",")
      (fields(0).toLong, fields(1))
    }
    val ccByUsername = users.join(cc).map {
      case (id, (username, cc)) => (username, cc)
    }
    // Print the result
    println(ccByUsername.collect().mkString("\n"))
  }
}
```

## 三角形计算算法

在图中，如果一个顶点有两个邻接顶点并且顶点与顶点之间有边相连，那么我们就可以把三个顶点归于一个三角形。

这里通过计算社交网络图中三角形的数量，所采用的数据集同样和PageRank的数据集一样。

```
import org.apache.log4j.{Level,Logger}
import org.apache.spark._
import org.apache.spark.graphx.{GraphLoader,PartitionStrategy}
object SimpleGraphX {
  def main(args: Array[String]) {
    //屏蔽日志
    Logger.getLogger("org.apache.spark").setLevel(Level.WARN)
    Logger.getLogger("org.eclipse.jetty.server").setLevel(Level.OFF)
    //设置运行环境
    val conf = new SparkConf().setAppName("SimpleGraphX").setMaster("local")
    val sc = new SparkContext(conf)
    // Load the edges in canonical order and partition the graph for triangle count
    val graph = GraphLoader.edgeListFile(sc, "file:///usr/local/Spark/data/graphx/followers.txt", true)
      .partitionBy(PartitionStrategy.RandomVertexCut)
    // Find the triangle count for each vertex
    val triCounts = graph.triangleCount().vertices
    // Join the triangle counts with the usernames
    val users = sc.textFile("file:///usr/local/Spark/data/graphx/users.txt").map { line =>
      val fields = line.split(",")
      (fields(0).toLong, fields(1))
    }
    val triCountByUsername = users.join(triCounts).map { case (id, (username, tc)) =>
      (username, tc)
    }
    // Print the result
    println(triCountByUsername.collect().mkString("\n"))
  }
}
```