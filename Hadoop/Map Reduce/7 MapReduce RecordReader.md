# MapReduce RecordReader

为了理解 MapReduce 框架的 RecordReader 原理，首先得搞清楚 Hadoop 的数据流程。下面我来了解一下 Hadoop 里面的数据流程是怎样的。

![mapreduce recordreader](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570245251000_20191005111414251628.png)

## Hadoop RecordReader 简介

MapReduce 有一个简单数据处理模型，map 和 reduce 函数的输入数据和输出数据都必须是键值对（key-value pairs）。Hadoop MapReduce 的 map 和 Reduce 函数具有以下通用的形式：

- **map**：(K1, V1) —> list(K2, V2)
- **reduce**：(K2, list(V2)) —> list(K3, V3)

在 MapReduce 运行之前，它必须要知道到底需要处理什么数据，这个过程由 InputFormat 类来完成，InputFormat 类主要负责从 HDFS 读取 map 函数需要处理的文件，并且它还会创建 InputSplit 并把 InputSplit 转换成记录。在 HDFS 数据会被分割成若干个块，块大小默认为 128 MB。默认每个分片对应一个块，由一个 map 任务处理。

InputFormat 类调用 `getSplits()` 函数计算每个文件的分片，然后把分片发送给 JobTracker，它利用分片的存储位置信息，把执行该分片的 map 任务调度到 TaskTracker。然后，Map 任务把分片传递给 TaskTracker 里面的 InputFormat类的 `createRecordReader()` 方法，以便获取分片的 RecordReader 对象。RecordReader 读取分片数据并把它转换成适合 mapper 读取的键值对形式。

Hadoop RecordReader 读取由 InputSplit 创建的数据并创建相应的键值对。“start” 是 RecordReader 开始生成键值对的起始字节位置，也就是一条记录的起始位置，“end” 是停止读取记录的标志，一条记录的结束位置。这里的 “start” 和 “end” 是由 InputSplit 确定的，“start” 通过 `InputSplit.getStart()` 获取，“end” 通过 `InputSplit.getStart + InputSplit.length` 获取。

## RecordReader 工作原理

RecordReader 不仅仅是记录的迭代器。map 任务利用一个记录生成一个键值对，我们可以通过 mapper 的 run 函数验证这一点：

```
public void run(Context context) throws IOException, InterruptedException{    setup(context);    while(context.nextKeyValue())    {        map(context.getCurrentKey(),context.getCurrentValue(),context)    }    cleanup(context);}
```

在执行 `setup()` 之后， `nextKeyValue()` 将重复从 context 获取下个记录来填充 mapper 的 key 和 value 对象。而 key 和 value 是通过 context 读取的。map 函数的输入数据是键值对形式的 (K, V)，当读取记录结束时，nextKeyValue 方法将返回 false。

RecordReader 根据 InputSplit 对数据划分好的数据边界进行读取，并生成键值对，但这个并非强制的，你可以通过自定义 RecordReader 来读取更多 InputSplit 之外的数据，但并不鼓励这么做。

## RecordReader 分类

RecordReader 实例是被 InputFormat 定义的，默认情况下，它使用 TextInputFormat 把数据转换成键值对。TextInputFormat 提供 2 种 RecordReader，分别是 LineRecordReader 和 SequenceFileRecordReader。

### LineRecordReader

LineRecordReader 是 TextInputFormat 默认的 RecordReader。它把输入文件的每一行内容作为 value，字节偏移量作为 key。

### SequenceFileRecordReader

这个是 SequenceFileInputFormat 对应的 RecordReader。用于把二进制数据转换成记录形式。

## 单个记录的最大值

一个被处理的记录的大小是有限制的，默认的最大值为 Integer.MAX_VALUE。可以通过下面参数设置这个最大值：

```
conf.setInt("mapred.linerecordreader.maxlength", Integer.MAX_VALUE);
```

如果一行数据的大小超过这个最大值，那么该记录会被忽略掉。

