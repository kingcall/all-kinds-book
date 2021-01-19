# MapReduce OutputFormat

Hadoop OutputFormat 负责检验 job 的输出规范，RecordWriter 把输出数据写到输出文件的具体实现就是由 OutputFormat 决定的。

![mapreduce outputformat](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570246489000_20191005113450740650-20210112084933691.png)

## Hadoop Outputformat

在开始学习 MapReduce 框架的 OutputFormat 之前，让我们先来看一下 RecordWriter ，以及它在 MapReduce 框架起到什么样的作用。

### Hadoop RecordWriter

我们知道，Reducer 以 mapper 输出的键值对结果为输入数据，并对这些数据执行 reducer 函数，它最终输出的结果依然是键值对数据。RecordWriter 负责把 Reducer 输出的键值对数据写到输出文件。

### Hadoop Outputformat

在前面我们已经了解到，Hadoop RecordWriter 获取 Reducer 输出的数据，并把这些数据写到输出文件。而 RecordWriter 用怎样的格式把数据写到文件，这些行为是由 OutputFormat 决定的。OutputFormat 和 InputFormat 函数两者之间有许多共同之处。OutputFormat 函数负责把文件写到 HDFS 或者本地磁盘，它描述了 MapReduce job 输出数据的规范：

- MapReduce job 检查输出目录是否已经存在
- OutputFormat 提供的 RecordWriter 把结果数据写到输出文件，该输出文件会被存储在文件系统。

**`FileOutputformat.setOutputPath()`** 方法用来设置输出目录。每个 Reducer 会把数据写到公共目录下的一个独立文件中，每个 reducer 对应一个文件。

## Hadoop OutputFormat 的分类

和 InputFormat 一样，Hadoop OutputFormat 也分好几种不同分类。

### TextOutputFormat

Reducer 默认的输出格式是 TextOutputFormat，它是以每一行一个键值对的形式写入数据的，key 和 value 可能是任意数据类型的，但最终 TextOutputFormat 会调用 `toString()` 方法把他们转换成字符串类型。键值对之间以 tab 键作为分割符，你可以通过修改下面的属性修改这个默认的设置：

```
MapReduce.output.textoutputformat.separator
```

与 TextOutputFormat 对应的输入格式是 KeyValueTextInputFormat，它通过可配置的分隔符将键值对文本行分割。

### SequenceFileOutputFormat

SequenceFileOutputformat 将它的输出写为一个顺序文件。如果输出需要作为后续 MapReduce 任务的输入，这是一种比较好的输出格式，因为它的格式紧凑，很容易被压缩。压缩由 SequenceFileOutputformat 的静态方法 `putCompressionType()` 来实现。

### SequenceFileAsBinaryOutputFormat

SequenceFileAsBinaryOutputFormat 与 SequenceFileAsBinaryInputFormat 相对应，它以原始的二进制格式把键值对写到一个顺序文件中。

### MapFileOutputFormat

MapFileOutputformat 以 map 文件作为输出。MapFile 中的键必须顺序添加，所以必须确保 reducer 输出的键是已经拍好序的。

### MultipleOutput

MultipleOutput 类可以将数据写到多个文件，这些文件的名称源于输出的键和值或者任意字符串。这允许每个 reducer（或者只有 map 作业的 mapper ）创建多个文件。采用 name-m-nnnnn 形式的文件名用于 map 输出，name-r-nnnnn 形式的文件名用于 reduce 输出，其中 name 是由程序设定的任意名字，nnnnn 是一个指明块号的整数（从 00000 开始）。块号保证从不同分区（mapper 或者 reducer）写的输出在相同名字情况下不会冲突。

### LazyOutputFormat

FileOutputformat 的子类会产生输出文件（ part-r-nnnnn ），即使文件是空的。有些应用倾向于不创建空文件，此时 LazyOutputFormat 就有用武之地了。它是一个封装输出格式，可以保证指定分区第一条记录输出时才真正创建文件。要使用它，用 JobConf 和相关的输出格式作为参数来调用 `setOutputFormatClass()` 方法即可。

### DBOutputFormat

它适用于将作业输出数据转储到关系数据库和 HBase。