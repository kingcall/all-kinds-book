# MapReduce InputFormat

Hadoop InputFormat 会检查作业的数据输入规范，它把输入文件分割成 InputSplit 分片，并发送给 Mapper。

![mapreduce inputformat](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570244539000_20191005110223024405.png)

## Hadoop InputFormat

输入文件的分片和读取是由 InputFormat 定义的。InputFormat 主要负责创建数据分片，并把它转换成记录（即键值对），如果你还不熟悉 MapReduce 作业的工作原理，请参考 [MapReduce 工作原理](http://www.hadoopdoc.com/mapreduce/mapreduce-theory)。

MapReduce 任务处理的数据是存储在输入文件的，而输入文件一般保存在 HDFS。虽然这些文件的格式可以是任意的，如基于行的日志文件和二进制格式都有可能被使用。利用 InputFormat 我们可以分割和读取这些文件。InputFormat 类是 MapReduce 框架最基础的类之一，它提供如下功能：

- 数据切分，按照某个策略将输入数据且分成若干个 split，以便确定 Map Task 的个数即 Mapper 的个数，在 MapReduce 框架中，一个 split 就意味着需要一个 Map Task。
- 为 Mapper 提供输入数据，即给定一个 split，使用其中的 RecordReader 对象将之解析为一个个的key/value 键值对。

## 如何把数据发送给 Mapper

MapReduce 框架有 2 种方法可以把数据发送给 `mapper:getsplits()` 和 `createRecordReader()` 。

```
public abstract class InputFormat<K, V>{    public abstract List<InputSplit> getSplits(JobContext context)    throws IOException, InterruptedException;    public abstract RecordReader<K, V> createRecordReader(InputSplit split,    TaskAttemptContext context) throws IOException,    InterruptedException;}
```

## MapReduce InputFormat 的几种类型

![MapReduce InputFormat 的几种类型](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570244784000_20191005110626015450.png)

让我们来看一下 Hadoop 里面的几种 InputFormat。

### FileInputFormat

它是所有基于文件的 InputFormat 的基类。Hadoop FileInputFormat 指定数据文件所在的输入目录。当我们启动一个 Hadoop 作业的时候，FileInputFormat 提供了一个包含要读取的文件的路径，它会读取路径下的所有文件，并把这些文件分割成一个或多个 InputSplit 。

### TextInputFormat

这个是 MapReduce 默认使用的 InputFormat。TextInputFormat 会把输入文件的每一个数据行当成一个记录，这对于无格式数据或者基于行数据文件（如日志文件）非常有用。

### KeyValueTextInputFormat

KeyValueTextInputFormat 和 TextInputFormat 类似，它也会把输入文件的每一个数据行当成一个记录。TextInputFormat 是把数据行的内容作为键值对的 Key 部分，而 KeyValueTextInputFormat 分割 Key 和 value 是用 tab 键（‘\t’）来区分的。在一行数据里面，tab 键前面的内容作为 Key，tab 键后面的内容作为 Value。

### SequenceFileInputFormat

Hadoop SequenceFileInputFormat 用来读取顺序文件（sequence file）的。sequence 文件是一种存储二进制键值对序列的二进制文件。sequence 文件是块压缩的，并且提供了几种数据类型（不仅仅是文本类型）直接的序列化和反序列化操作。sequence 文件可以作为 MapReduce 任务的输出数据，并且用它做一个 MapReduce 作业到另一个作业的中间数据是很高效的。

### SequenceFileAsTextInputFormat

Hadoop SequenceFileAsTextInputFormat 是 SequenceFileInputFormat 的另一种形式，它把 sequence 文件的 key / value 转换成文本对象。通过调用 “toString()” 方法，就可以把 key / value 转换成文本对象（Text Object）。这种 InputFormat 使得 sequence 文件适合作为流输入。

### SequenceFileAsBinaryInputFormat

跟 SequenceFileAsTextInputFormat 类似，只是它从 sequence 文件读取 Key / Value 的时候是以二进制的形式读取的。

### NLineInputFormat

Hadoop NLineInputFormat 是 TextInputFormat 的另一种形式，key 是数据行的字节偏移量，value 是数据行具体内容。如果InputFormat 是 TextInputFormat 和 KeyValueTextInputFormat，那么每个 mapper 接收的输入行数是可变的，并且 mapper 的数量取决于分片的数量和数据行的长度。如果我们想要让 mapper 接收固定数量的输入行的话，我们可以使用 NLineInputFormat 。NLineInputFormat 前面的 N 表示每个Mapper收到输入的行数。默认 情况下（N=1），每个 mapper 刚好接收一行，如果 N = 2，那么每个分片是 2 行，一个 mapper 将接收两个键值对。

### DBInputFormat

Hadoop DBInputFormat 是一种使用 JDBC 从关系数据库读取数据的 InputFormat。使用 DBInputFormat 读取数据的时候， Key 是 LongWritables 类型，而 Value 是 DBWritables 类型。