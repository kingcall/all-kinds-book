# Spark GraphX 图操作

在GraphX中，核心操作都是被优化过的，组合核心操作的定义在GraphOps中。

由于Scala隐式转换，定义在GraphOps的操作可以在Graph的成员中获取。例如：我们计算图中每个顶点的入度.(该方法是定义在GraphOps)

```
val graph: Graph[(String, String), String]
// Use the implicit GraphOps.inDegrees operator
val inDegrees: VertexRDD[Int] = graph.inDegrees
```

下面我们列出常用的几个图操作。

## 操作列表概述

这里只列出Graph中常用的操作函数API，仍有一些高级函数没有列出，如果需要还请参考Spark API文档。

```
/** Summary of the functionality in the property graph */
class Graph[VD, ED] {
  // Information about the Graph ===================================================================
  val numEdges: Long
  val numVertices: Long
  val inDegrees: VertexRDD[Int]
  val outDegrees: VertexRDD[Int]
  val degrees: VertexRDD[Int]
  // Views of the graph as collections =============================================================
  val vertices: VertexRDD[VD]
  val edges: EdgeRDD[ED]
  val triplets: RDD[EdgeTriplet[VD, ED]]
  // Functions for caching graphs ==================================================================
  def persist(newLevel: StorageLevel = StorageLevel.MEMORY_ONLY): Graph[VD, ED]
  def cache(): Graph[VD, ED]
  def unpersistVertices(blocking: Boolean = true): Graph[VD, ED]
  // Change the partitioning heuristic  ============================================================
  def partitionBy(partitionStrategy: PartitionStrategy): Graph[VD, ED]
  // Transform vertex and edge attributes ==========================================================
  def mapVertices[VD2](map: (VertexId, VD) => VD2): Graph[VD2, ED]
  def mapEdges[ED2](map: Edge[ED] => ED2): Graph[VD, ED2]
  def mapEdges[ED2](map: (PartitionID, Iterator[Edge[ED]]) => Iterator[ED2]): Graph[VD, ED2]
  def mapTriplets[ED2](map: EdgeTriplet[VD, ED] => ED2): Graph[VD, ED2]
  def mapTriplets[ED2](map: (PartitionID, Iterator[EdgeTriplet[VD, ED]]) => Iterator[ED2])
    : Graph[VD, ED2]
  // Modify the graph structure ====================================================================
  def reverse: Graph[VD, ED]
  def subgraph(
      epred: EdgeTriplet[VD,ED] => Boolean = (x => true),
      vpred: (VertexId, VD) => Boolean = ((v, d) => true))
    : Graph[VD, ED]
  def mask[VD2, ED2](other: Graph[VD2, ED2]): Graph[VD, ED]
  def groupEdges(merge: (ED, ED) => ED): Graph[VD, ED]
  // Join RDDs with the graph ======================================================================
  def joinVertices[U](table: RDD[(VertexId, U)])(mapFunc: (VertexId, VD, U) => VD): Graph[VD, ED]
  def outerJoinVertices[U, VD2](other: RDD[(VertexId, U)])
      (mapFunc: (VertexId, VD, Option[U]) => VD2)
    : Graph[VD2, ED]
  // Aggregate information about adjacent triplets =================================================
  def collectNeighborIds(edgeDirection: EdgeDirection): VertexRDD[Array[VertexId]]
  def collectNeighbors(edgeDirection: EdgeDirection): VertexRDD[Array[(VertexId, VD)]]
  def aggregateMessages[Msg: ClassTag](
      sendMsg: EdgeContext[VD, ED, Msg] => Unit,
      mergeMsg: (Msg, Msg) => Msg,
      tripletFields: TripletFields = TripletFields.All)
    : VertexRDD[A]
  // Iterative graph-parallel computation ==========================================================
  def pregel[A](initialMsg: A, maxIterations: Int, activeDirection: EdgeDirection)(
      vprog: (VertexId, VD, A) => VD,
      sendMsg: EdgeTriplet[VD, ED] => Iterator[(VertexId,A)],
      mergeMsg: (A, A) => A)
    : Graph[VD, ED]
  // Basic graph algorithms ========================================================================
  def pageRank(tol: Double, resetProb: Double = 0.15): Graph[Double, Double]
  def connectedComponents(): Graph[VertexId, ED]
  def triangleCount(): Graph[Int, ED]
  def stronglyConnectedComponents(numIter: Int): Graph[VertexId, ED]
}
```

## 属性操作

属性图中包括类似RDD map的操作，如下图：

```
class Graph[VD, ED] {
  def mapVertices[VD2](map: (VertexId, VD) => VD2): Graph[VD2, ED]
  def mapEdges[ED2](map: Edge[ED] => ED2): Graph[VD, ED2]
  def mapTriplets[ED2](map: EdgeTriplet[VD, ED] => ED2): Graph[VD, ED2]
}
```

mapVertices遍历所有的顶点，mapEdges遍历所有的边，mapTriplets遍历所有的三元组。

注意，属性操作下，图的结构都不受影响。这些操作的一个重要特征是它允许所得图形重用原有图形的结构索引(indices)。

属性操作常用来进行特殊计算或者排除不需要的属性.我们依旧以上一章节的图为例，进行操作。下面没有列出全部代码，包的导入和图的构建请参考上一章节的内容 Spark GraphX - 简介 。

```
/***********************************
*属性操作
***********************************/
 println("---------------------------------------------")
 println("给图中每个顶点的职业名的末尾加上'dblab'字符串")
 graph.mapVertices{ case (id, (name, occupation)) => (id, (name, occupation+"dblab"))}.vertices.collect.foreach(v => println(s"${v._2._1} is ${v._2._2}"))
 println("---------------------------------------------")
 println("给图中每个元组的Edge的属性值设置为源顶点属性值加上目标顶点属性值：")
 graph.mapTriplets(triplet => triplet.srcAttr._2 + triplet.attr + triplet.dstAttr._2).edges.collect.foreach(println(_))
```

## 结构操作

目前Spark GraphX只支持一些简单的常用结构操作，还在不断完善中。

常用的操作如下：

```
class Graph[VD, ED] {
  def reverse: Graph[VD, ED]
  def subgraph(epred: EdgeTriplet[VD,ED] => Boolean,
               vpred: (VertexId, VD) => Boolean): Graph[VD, ED]
  def mask[VD2, ED2](other: Graph[VD2, ED2]): Graph[VD, ED]
  def groupEdges(merge: (ED, ED) => ED): Graph[VD,ED]
}
```

reverse操作返回一个所有边方向取反的新图。该反转操作并没有修改图中顶点、边的属性，更没有增加边的数量。

subgraph操作主要利用顶点和边进行判断，返回的新图中包含满足判断要求的顶点、边。该操作常用于一些情景，比如：限制感兴趣的图顶点和边，删除损坏连接。如下代码：

```
/***********************************
*展示结构操作
***********************************/
graph.triplets.map(
   triplet => triplet.srcAttr._1 + " is the " + triplet.attr + " of " + triplet.dstAttr._1
).collect.foreach(println(_))
println("---------------------------------------------")
println("删除不存在的节点，构建子图")
val validGraph = graph.subgraph(vpred = (id, attr) => attr._2 != "Missing")
validGraph.vertices.collect.foreach(println(_))
validGraph.triplets.map(
   triplet => triplet.srcAttr._1 + " is the " + triplet.attr + " of " + triplet.dstAttr._1
).collect.foreach(println(_))
println("---------------------------------------------")
println("构建职业是professor的子图,并打印子图的顶点")
val subGraph = graph.subgraph(vpred = (id, attr) => attr._2 == "prof")
subGraph.vertices.collect.foreach(v => println(s"${v._2._1} is ${v._2._2}"))
```

mask 操作构造一个子图，这个子图包含输入图中包含的顶点和边。这个操作可以和subgraph操作相结合，基于另外一个相关图的特征去约束一个图。例如，我们可以使用丢失顶点的图运行连通分支，然后限制有效子图的返回,见如下代码：

```
println("---------------------------------------------")
println("运行联通分支")
val ccGraph = graph.connectedComponents() 
val validCCGraph = ccGraph.mask(validGraph)

```

groupEdges 操作合并多重图中的并行边(如顶点对之间重复的边)。在大量的 应用程序中，并行的边可以合并(它们的权重合并)为一条边从而降低图的大小。

## 关联操作

在很多情况下，需要将外部数据添加到图中。例如，我们可能有额外的用户属性，我们想把它融合到一个存在图中或者从一个图提取数据属性到另一个图。这些任务可以使用join操作来实现。下面我们列出了关键的join操作：

```
class Graph[VD, ED] {
  def joinVertices[U](table: RDD[(VertexId, U)])(map: (VertexId, VD, U) => VD)
    : Graph[VD, ED]
  def outerJoinVertices[U, VD2](table: RDD[(VertexId, U)])(map: (VertexId, VD, Option[U]) => VD2)
    : Graph[VD2, ED]
}

```

joinVertices操作连接外部RDD的顶点，返回一个新的带有顶点特征的图。这些特征是通过在连接顶点的结果上使用用户自定义的 map 函数获得的。没有 匹配的顶点保留其原始值。

outerJoinVertices操作和joinVertices操作相似,但用户自定义的map函数可以被应用到所有顶点和改变顶点类型。

```
/***********************************
 *展示关联操作
***********************************/
println("**********************************************************")
println("关联操作")
println("**********************************************************")
val inDegrees: VertexRDD[Int] = graph.inDegrees
case class User(name: String, occupation: String, inDeg: Int, outDeg: Int)
//创建一个新图，顶点VD的数据类型为User，并从graph做类型转换
val initialUserGraph: Graph[User, String] = graph.mapVertices { case (id, (name,   occupation)) => User(name, occupation , 0, 0)}
//initialUserGraph与inDegrees、outDegrees（RDD）进行连接，并修改initialUserGraph中inDeg值、outDeg值
val userGraph = initialUserGraph.outerJoinVertices(initialUserGraph.inDegrees) {
      case (id, u, inDegOpt) => User(u.name, u.occupation, inDegOpt.getOrElse(0), u.outDeg)
    }.outerJoinVertices(initialUserGraph.outDegrees) {
      case (id, u, outDegOpt) => User(u.name, u.occupation, u.inDeg,outDegOpt.getOrElse(0))
    }
println("连接图的属性：")
userGraph.vertices.collect.foreach(v => println(s"${v._2.name} inDeg: ${v._2.inDeg}  outDeg: ${v._2.outDeg}"))
println("出度和入度相同的人员：")
userGraph.vertices.filter {
   case (id, u) => u.inDeg == u.outDeg
 }.collect.foreach {
   case (id, property) => println(property.name)
}
```

## 聚合操作

在很多图分析任务中一个关键步骤就是集合每一个顶点的邻居信息。例如，我们想知道每一个用户的追随者数量或者追随者的平均年龄。一些迭代的图算法（像PageRank,最短路径和联通组件）就需要反复得聚合相邻顶点的属性。

### 信息聚合(aggregateMessages)

在GraphX中最核心的聚合操作就是aggregateMessages。它主要功能是向邻边发消息，合并邻边收到的消息。

```
class Graph[VD, ED] {
  def aggregateMessages[Msg: ClassTag](
      sendMsg: EdgeContext[VD, ED, Msg] => Unit,
      mergeMsg: (Msg, Msg) => Msg,
      tripletFields: TripletFields = TripletFields.All)
    : VertexRDD[Msg]
}
```

接口中含有三个参数，分别表示：

sendMsg:发消息函数
mergeMsg:合并消息函数
tripletFields:发消息的方向

下面使用aggregateMessages来计算比用户年龄更大的追随者的平均年龄，这里我们新建一个模型图代替上一节创建的模型的图，整体代码如下：

```
import org.apache.log4j.{Level,Logger}
import org.apache.spark._
import org.apache.spark.graphx._
import org.apache.spark.graphx.util.GraphGenerators
object SimpleGraphX {
  def main(args: Array[String]) {
    //屏蔽日志
    Logger.getLogger("org.apache.spark").setLevel(Level.WARN)
    Logger.getLogger("org.eclipse.jetty.server").setLevel(Level.OFF)
    //设置运行环境
    val conf = new SparkConf().setAppName("SimpleGraphX").setMaster("local")
    val sc = new SparkContext(conf)
    val graph: Graph[Double, Int] =
      GraphGenerators.logNormalGraph(sc, numVertices = 100).mapVertices( (id, _) => id.toDouble )
    // Compute the number of older followers and their total age
    val olderFollowers: VertexRDD[(Int, Double)] = graph.aggregateMessages[(Int, Double)](
      triplet => { // Map Function
        if (triplet.srcAttr > triplet.dstAttr) {
          // Send message to destination vertex containing counter and age
          triplet.sendToDst(1, triplet.srcAttr)
        }
      },
      // Add counter and age
      (a, b) => (a._1 + b._1, a._2 + b._2) // Reduce Function
    )
    val avgAgeOfOlderFollowers: VertexRDD[Double] =
      olderFollowers.mapValues( (id, value) =>
        value match { case (count, totalAge) => totalAge / count } )
    avgAgeOfOlderFollowers.collect.foreach(println(_))
  }
```

## 计算每个顶点的度

在上一节中，就已经用了计算每个顶点的度(每一个顶点边的数量)的实例，这也是一种常见的聚合操作。在有向图的情况下，它经常知道入度，出度和每个顶点的总度。 GraphOps 类包含了每一个顶点的一系列的度的计算。例如：在下面将计算最大入度，出度和总度：

```
def max(a: (VertexId, Int), b: (VertexId, Int)): (VertexId, Int) = {
  if (a._2 > b._2) a else b
}
val maxInDegree: (VertexId, Int)  = graph.inDegrees.reduce(max)
val maxOutDegree: (VertexId, Int) = graph.outDegrees.reduce(max)
val maxDegrees: (VertexId, Int)   = graph.degrees.reduce(max)

```

## 缓存操作

Spark中，RDDs默认是没有持久存储在内存。当多次使用RDDs时，为了避免重复计算,RDDs必须被显式缓存。GraphX中的图也是相同的方式,当使用一个图多次时，首先确认调用`Graph.cache()`。

对于迭代计算来说，迭代的中间结果将填充到缓存中。虽然最终会被删除，但是保存在内存中的不需要的数据将会减慢垃圾回收。对于迭代计算，我们建议使用 Pregel API，它可以正确的不持久化中间结果。