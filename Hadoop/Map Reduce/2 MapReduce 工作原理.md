# MapReduce 工作原理

本节主要详细介绍 MapReduce 的工作原理。
![mapreduce 工作原理](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/clipboard_20191004215521283279.png)

上面这个流程图已经把 MapReduce 的工作过程说的很清楚了，下面我们来一个一个拆解一下。

## 输入文件

首先，MapReduce 任务的目的是处理数据，那数据从哪里来？一般一个 MapReduce 任务的输入数据是来自于 HDFS 文件，这里的数据文件就叫做 MapReduce 任务的输入文件，而 HDFS 上文件的格式多种多样，比如有文本文件，二进制文件等。

## InputFormat

InputFormat 是 MapReduce 框架的一个类，它对输入文件进行分割和读取，并创建数据分片 InputSplit。

## InputSplit

InputSplit 对象即数据分片对象，由 InputFormat 生成的，一个数据分片由一个 Mapper 来处理，数据分片是逻辑上的划分，并非物理分割。每一个分片都会相应创建一个 map 任务，因此，map 任务的数量等于分片的数量，即有多少个分片就有多少个 map 任务。分片会被划分成记录，并且每个记录都会被对应 mapper 处理。

## RecordReader

它会跟 InputSplit 交互，并把数据转换成适合 mapper 读取的键值对（key-value pair）记录。默认情况下，它用的是 TextInputFormat 类来做转换。RecordReader 与 InputSplit 交互一直到文件读取完成。它会给文件的每一行数据分配一个字节偏移量（byte offset）作为唯一编号。后续这些键值对将被发送给 mapper 做进一步处理。

## Mapper

它负责处理每一个来自 RecordReader 的记录，并生成新的键值对数据，这些 Mapper 新生成的键值对跟输入键值对是不一样的。Mapper 的输出也就是我们前面说的中间结果将会被写到本地磁盘。Mapper 的输出数据并不是存储在 HDFS 的，因为这是临时数据，如果把临时数据写到 HDFS ，将造成不必要的复制，会导致 map 任务性能低下。Mapper 的输出数据被传输给 Combiner 做下一步处理。

## Combiner

combiner 其实是一种 reduce 操作。它会对 mapper 的输出数据做本地聚合，也就是说它是在输出数据的 mapper 所在的机器上执行的。主要为了减少 mapper 和 reducer 之间的数据传输。combiner 执行完成之后，它的输出结果就会被传输到 partitioner 做下一步处理。

## Partitioner

如果一个 MapReduce 作业在 reduce 阶段有多个 reducer 任务参与，才会有 Partitioner 这一步，即数据分区。如果只有一个 reducer 任务，Partitioner 是不会执行的，即不会对数据分区。

Partitioner 对来自 combiner 的输出数据分区并排序，其实就是对数据的 key 做哈希运算，具有相同 key 的记录会被分到相同的分区，然后每个分区会被发送给 reducer。

## Shuffle 和排序

现在，Partitioner 的输出被 shuffle 到 reduce 节点（ 这里的 reduce 节点其实就是正常的 slave 节点，由于在上面跑 reduce 任务所以才叫 reduce 节点）。shuffle 是对数据进行跨网络的物理移动，需要消耗网络带宽资源。在所有 mapper 都完成之后，他们的输出数据才会被 shuffle 到 reduce 节点，并且这些 mapper 产生的数据会被合并和排序，然后作为 reduce 阶段的输入数据。

## Reducer

在 reduce 阶段，它把 mapper 输出的键值对数据作为输入，然后对每个键值对数据记录应用 reducer 函数并输出结果。reducer 的输出数据是 MapReduce 作业的最终计算结果，它会被存储到 HDFS。

## RecordWrite

它负责把来自 Reducer 输出的键值对数据写到输出文件。

## OutputFormat

RecordWriter 将 Reducer 输出的键值对写入输出文件的方式由 OutputFormat 决定。OutputFormat 是由 Hadoop 提供的用于把数据写到 HDFS 或者本地磁盘的接口。因此，reducer 的最终输出数据是由 Outputformat 实例负责写入到 HDFS 的。

以上就是 MapReduce 完整的工作流程了。后续的教程会对每个步骤进行详细分析。