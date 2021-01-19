# MapReduce Mapper

hadoop mapper 任务主要负责处理每个输入记录，并生成一个新 键值对，这个 键值对跟输入记录是完成不一样的。mapper 任务的输出数据由这些 键值对组成的集合。在 mapper 任务把数据写到本地磁盘之前，数据会被按 key 进行分区并排序，分区的目的是要把 key 相同的值聚集在一起。

MapReduce 框架为每个 InputSplit（数据分片）生成一个 map 任务，这里的 InputSplit 是由 InputFormat 生成的。

mapper 只会处理键值对形式的数据，所以把数据传给 mapper之前，需要把它转换成 这样的格式。

![Hadoop mapper](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570243872000_20191005105116161957-20210112084151092.png)

## 键值对是怎么生成的

让我们详细的来讨论一下键值对 是怎么生成的。

**InputSplit** ：它是数据的逻辑表示，即数据分片，每个 InputSplit 会被一个 map 任务处理。
**RecordReader** ：它会跟 InputSplit 交互，并把数据转换成适合 mapper 读取的键值对（key-value pair）记录。默认情况下，它用的是 TextInputFormat 类来做转换。RecordReader 与 InputSplit 交互一直到文件读取完成。它会给文件的每一行数据分配一个字节偏移量（byte offset）作为唯一编号。后续这些键值对将被发送给 mapper 做进一步处理。

## Hadoop Mapper 是如何工作的

下面让我们来看一下 hadoop 里面 mapper 的工作过程。

InputSplit 会为 mapper 将 HDFS 块转换成逻辑分片。比如，要读取的文件是 200MB，那么就需要 2 个 InputSplit，因为这个有文件有 2 个block（默认块大小为 128MB），MapReduce 会为每个块创建一个 InputSplit，然后为每个 InputSplit 创建一个 RecordReader 和一个 mapper。

InputSplit 的数量并非总是依赖于数据块的数量，我们可以通过设置下面的属性，来为文件指定分片的数量。

```
mapred.max.split.size
```

RecordReader 主要负责读取数据，并把数据转换成 的形式。它会为文件的每一行数据分配一个字节偏移量（唯一编号），这些键值对数据会被发送给 mapper，mapper 处理之后把数据落地到本地磁盘，我们把 mapper 输出的数据叫做中间数据，其实就是临时数据。

## map 任务的数量

本小节我们来讨论一下在执行 MapReduce 作业的时候，对于给定的数据量，如何计算出 mapper 任务数量？一般情况下，输入文件的数据块总量决定了 mapper 任务的数量。对于 map 阶段，每个节点的最佳并行度在 10 到 100 个 mapper 之间，虽然对于非 CPU 密集型的 map 任务，并行度已经被设置为 300 个 map。由于任务的启动需要消耗一些时间，所以如果 map 的执行超过一分钟的话，任务运行效率会更高。

举个例子，假如 HDFS 的块大小是 128MB，需要处理的数据的大小是 10TB，那么就需要 82000 个 map。这个 map 的数量由 InputFormat 类决定，其计算公式如下：

```
map 任务数量 = {( 数据总容量 ) / ( 分片大小 )}
```

如果数据是 1TB，数据分片大小是 100MB 的话，那么 map 任务数 = ( 1000 * 1000 ) / 100 = 10000。即 10000 个 map。