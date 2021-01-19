# Spark Streaming DStream

## DStream

即**Discretized Stream**，中文叫做**离散流**，Spark Streaming提供的一种高级抽象，代表了一个持续不断的数据流。

DStream可以通过输入数据源来创建，比如Kafka、Flume，也可以通过对其他DStream应用高阶函数来创建，比如map、reduce、join、window。

DStream的内部，其实是一系列持续不断产生的RDD，RDD是Spark Core的核心抽象，即不可变的，分布式的数据集。

DStream中的每个RDD都包含了一个时间段内的数据。
![spark streaming 离散数据流](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1571323597000_20191017224640778218.png)

对DStream应用的算子，其实在底层会被翻译为对DStream中每个RDD的操作，比如对一个DStream执行一个map操作，会产生一个新的DStream，其底层原理为，对输入DStream中的每个时间段的RDD，都应用一遍map操作，然后生成的RDD，即作为新的DStream中的那个时间段的一个RDD。

底层的RDD的transformation操作，还是由Spark Core的计算引擎来实现的，Spark Streaming对Spark core进行了一层封装，隐藏了细节，然后对开发人员提供了方便易用的高层次API。
![spark streaming DStream](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1571323643000_20191017224725834348.png)

## 操作DStream

对于从数据源得到的DStream，用户可以在其基础上进行各种操作。

与RDD类似，DStream也提供了自己的一系列操作方法，这些操作可以分成三类：普通的转换操作、窗口转换操作和输出操作。

### 普通的转换操作

| 转换                             | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| map(func)                        | 源 DStream的每个元素通过函数func返回一个新的DStream。        |
| flatMap(func)                    | 类似与map操作，不同的是每个输入元素可以被映射出0或者更多的输出元素。 |
| filter(func)                     | 在源DSTREAM上选择Func函数返回仅为true的元素,最终返回一个新的DSTREAM 。 |
| repartition(numPartitions)       | 通过输入的参数numPartitions的值来改变DStream的分区大小。     |
| union(otherStream)               | 返回一个包含源DStream与其他 DStream的元素合并后的新DSTREAM。 |
| count()                          | 对源DStream内部的所含有的RDD的元素数量进行计数，返回一个内部的RDD只包含一个元素的DStreaam。 |
| reduce(func)                     | 使用函数func（有两个参数并返回一个结果）将源DStream 中每个RDD的元素进行聚 合操作,返回一个内部所包含的RDD只有一个元素的新DStream。 |
| countByValue()                   | 计算DStream中每个RDD内的元素出现的频次并返回新的DStream[(K,Long)]，其中K是RDD中元素的类型，Long是元素出现的频次。 |
| reduceByKey(func, [numTasks])    | 当一个类型为（K，V）键值对的DStream被调用的时候,返回类型为类型为（K，V）键值对的新 DStream,其中每个键的值V都是使用聚合函数func汇总。注意：默认情况下，使用 Spark的默认并行度提交任务（本地模式下并行度为2，集群模式下位8），可以通过配置numTasks设置不同的并行任务数。 |
| join(otherStream, [numTasks])    | 当被调用类型分别为（K，V）和（K，W）键值对的2个DStream 时，返回类型为（K，（V，W））键值对的一个新 DSTREAM。 |
| cogroup(otherStream, [numTasks]) | 当被调用的两个DStream分别含有(K, V) 和(K, W)键值对时,返回一个(K, Seq[V], Seq[W])类型的新的DStream。 |
| transform(func)                  | 通过对源DStream的每RDD应用RDD-to-RDD函数返回一个新的DStream，这可以用来在DStream做任意RDD操作。 |
| updateStateByKey(func)           | 返回一个新状态的DStream,其中每个键的状态是根据键的前一个状态和键的新值应用给定函数func后的更新。这个方法可以被用来维持每个键的任何状态数据。 |

#### transform(func)

该transform操作（转换操作）及其类似的transformWith操作，允许在DStream上应用任意的RDD-to-RDD函数。它可以实现DStream API中未提供的操作，比如两个数据流的连接操作。

示例代码如下：

```
val spamInfoRDD = ssc.sparkContext.newAPIHadoopRDD(...) // RDD containing spam information
val cleanedDStream = wordCounts.transform { rdd =>
  rdd.join(spamInfoRDD).filter(...) // join data stream with spam information to do data cleaning
  ...
}
```

#### updateStateByKey操作

我们使用的一般操作都是不记录历史数据的，也就说只记录当前定义时间段内的数据，跟前后时间段无关。如果要统计历史时间内的总共数据并且实时更新，如何解决呢？该updateStateByKey操作可以让你保持任意状态，同时不断有新的信息进行更新。要使用**updateStateByKey操作**，必须进行下面两个步骤 ：

- 定义状态： 状态可以是任意的数据类型。
- 定义状态更新函数：用一个函数指定如何使用先前的状态和从输入流中获取的新值更新状态。

对DStream通过updateStateByKey(updateFunction)来实现实时更新。

更新函数有两个参数 ：

- newValues是当前新进入的数据。
- runningCount 是历史数据，被封装到了Option中。

为什么历史数据要封装到Option中呢？有可能我们没有历史数据，这个时候就可以用None，有数据可以用Some(x)。当然我们的当前结果也要封装到Some()中，以便作为之后的历史数据。

我们并不用关心新进入的数据和历史数据，系统会自动帮我们产生和维护，我们只需要专心写处理方法就行。

```
def updateFunction(newValues: Seq[Int], runningCount: Option[Int]): Option[Int] = {//定义的更新函数
    val newCount = ...  // add the new values with the previous running count to get the new count
    Some(newCount)
}
val runningCounts = pairs.updateStateByKey[Int](updateFunction)//应用
```

示例：

1. 首先我们需要了解数据的类型

2. 编写处理方法

3. 封装结果

   ```
   //定义更新函数
   //我们这里使用的Int类型的数据，因为要做统计个数
   def updateFunc(newValues : Seq[Int],state :Option[Int]) ：Some[Int] = {
    //传入的newVaules将当前的时间段的数据全部保存到Seq中
    //调用foldLeft(0)(_+_) 从0位置开始累加到结束   
    val currentCount = newValues.foldLeft(0)(_+_) 
    //获取历史值,没有历史数据时为None，有数据的时候为Some
    //getOrElse（x）方法，如果获取值为None则用x代替
    val  previousCount = state.getOrElse(0)
    //计算结果，封装成Some返回
    Some(currentCount+previousCount) 
   }
   //使用
   val stateDStream = DStream.updateStateByKey[Int](updateFunc)
   ```

### 窗口转换函数

Spark Streaming 还提供了窗口的计算，它允许你通过滑动窗口对数据进行转换，窗口转换操作如下：

| 转换                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| window(windowLength, slideInterval)                          | 返回一个基于源DStream的窗口批次计算后得到新的DStream。       |
| countByWindow(windowLength,slideInterval)                    | 返回基于滑动窗口的DStream中的元素的数量。                    |
| reduceByWindow(func, windowLength,slideInterval)             | 基于滑动窗口对源DStream中的元素进行聚合操作，得到一个新的DStream。 |
| reduceByKeyAndWindow(func,windowLength, slideInterval, [numTasks]) | 基于滑动窗口对（K，V）键值对类型的DStream中的值按K使用聚合函数func进行聚合操作，得到一个新的DStream。 |
| reduceByKeyAndWindow(func, invFunc,windowLength, slideInterval, [numTasks]) | 一个更高效的reduceByKkeyAndWindow()的实现版本，先对滑动窗口中新的时间间隔内数据增量聚合并移去最早的与新增数据量的时间间隔内的数据统计量。例如，计算t+4秒这个时刻过去5秒窗口的WordCount，那么我们可以将t+3时刻过去5秒的统计量加上[t+3，t+4]的统计量，在减去[t-2，t-1]的统计量，这种方法可以复用中间三秒的统计量，提高统计的效率。 |
| countByValueAndWindow(windowLength,slideInterval, [numTasks]) | 基于滑动窗口计算源DStream中每个RDD内每个元素出现的频次并返回DStream[(K,Long)]，其中K是RDD中元素的类型，Long是元素频次。与countByValue一样，reduce任务的数量可以通过一个可选参数进行配置。 |

在Spark Streaming中，数据处理是按批进行的，而数据采集是逐条进行的。因此在Spark Streaming中会先设置好批处理间隔（batch duration），当超过批处理间隔的时候就会把采集到的数据汇总起来成为一批数据交给系统去处理。

对于窗口操作而言，在其窗口内部会有N个批处理数据，批处理数据的大小由窗口间隔（window duration）决定，而窗口间隔指的就是窗口的持续时间，在窗口操作中，只有窗口的长度满足了才会触发批数据的处理。除了窗口的长度，窗口操作还有另一个重要的参数就是滑动间隔（slide duration），它指的是经过多长时间窗口滑动一次形成新的窗口，滑动窗口默认情况下和批次间隔的相同，而窗口间隔一般设置的要比它们两个大。

在这里必须注意的一点是滑动间隔和窗口间隔的大小一定得设置为批处理间隔的整数倍。
![spark streaming 窗口转换函数](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1571324031000_20191017225354872867.png)

如图所示，批处理间隔是1个时间单位，窗口间隔是3个时间单位，滑动间隔是2个时间单位。对于初始的窗口time 1-time 3，只有窗口间隔满足了定义的长度也就是3才触发数据的处理，不够3继续等待。当间隔满足3之后进行计算后然后进行窗口滑动，滑动2个单位，会有新的数据流入窗口。然后重复等待满足窗口间隔执行计算。

### 输出操作

Spark Streaming允许DStream的数据被输出到外部系统，如数据库或文件系统。由于输出操作实际上使transformation操作后的数据可以通过外部系统被使用，同时输出操作触发所有DStream的transformation操作的实际执行（类似于RDD操作）。

以下表列出了目前主要的输出操作：

| 转换                                    | 描述                                                         |
| --------------------------------------- | ------------------------------------------------------------ |
| **print**()                             | 在Driver中打印出DStream中数据的前10个元素。                  |
| **saveAsTextFiles**(prefix, [suffix])   | 将DStream中的内容以文本的形式保存为文本文件，其中每次批处理间隔内产生的文件以prefix-TIME_IN_MS[.suffix]的方式命名。 |
| **saveAsObjectFiles**(prefix, [suffix]) | 将DStream中的内容按对象序列化并且以SequenceFile的格式保存。其中每次批处理间隔内产生的文件以prefix-TIME_IN_MS[.suffix]的方式命名。 |
| **saveAsHadoopFiles**(prefix, [suffix]) | 将DStream中的内容以文本的形式保存为Hadoop文件，其中每次批处理间隔内产生的文件以prefix-TIME_IN_MS[.suffix]的方式命名。 |
| **foreachRDD**(func)                    | 最基本的输出操作，将func函数应用于DStream中的RDD上，这个操作会输出数据到外部系统，比如保存RDD到文件或者网络数据库等。需要注意的是func函数是在运行该streaming应用的Driver进程里执行的。 |

### DStream持久化

与RDD一样，DStream同样也能通过`persist()`方法将数据流存放在内存中。