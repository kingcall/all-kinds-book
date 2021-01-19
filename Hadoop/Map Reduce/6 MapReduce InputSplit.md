# MapReduce InputSplit

![mapreduce inputsplit原理](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570244985000_20191005110946428506-20210112084446572.png)

## Hadoop InputSplit

InputSplit 是数据的一种逻辑表示，即我们所说的文件的数据分片。每个分片由一个 mapper 处理，当然，mapper 并非直接对数据分片进行处理，在 mapper 处理数据分片之前，数据分片会被转换成记录，即键值对。mapper 直接操作键值对。

MapReduce InputSplit 的长度是以字节来度量的，每个 InputSplit 都有相应的存储位置（主机名称）。MapReduce 利用分片存储位置，把 map 任务调度到距离数据尽可能近的地方执行。Map 任务的执行优先级取决于分片的大小，分片大的最先执行，主要为了最小化作业的运行时间。**需要注意的是，InputSplit，即数据分片并不真正存储输入数据，它只是数据的一种引用，即一种数据的逻辑表示**。

我们并不需要直接处理 InputSplit，因为他们是由 InputFormat 创建的（ InputFormat 创建 InputSplit 并把它转换成键值对）。默认情况下，FileInputFormat 把文件分割成若干个块，每个块大小 128MB（和 HDFS 的块大小一样），这个默认值可以在配置文件 mapred-site.xml 修改，参数为 `mapred.min.split.size`。也可以在提交作业的时候修改该值。另外，可以定制化开发 InputFormat，来决定分割文件的方式。

## Hadoop 如何修改数据分片大小

我们可以在 MapReduce 程序根据数据的大小来调整分片大小。修改 `mapred.min.split.size` 这个参数即可。

客户端（运行 Job 的节点）调用 `getSplit()` 方法为一个作业计算分片大小，然后发送到 application master，它根据数据分片的存储位置对处理该数据分片的 map 任务进行调度。接下来 map 任务把分片传递给 InputFormat 的 `createRecordReader()` 方法以便获取分片的 RecordReader 对象，而 RecordReader 对象将生成记录，也就是键值对，然后传递给 map 函数。