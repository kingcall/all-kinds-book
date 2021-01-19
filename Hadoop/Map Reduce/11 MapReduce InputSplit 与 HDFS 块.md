# MapReduce InputSplit 与 HDFS 块

InputSplit 即数据分片，HDFS 块（block）即分布式存储系统的数据块概念。下面详细介绍这两个概念的区别和联系。

## HDFS 块与 InputSplit

### HDFS 块

块是硬盘上存储数据的一个连续位置。通常，文件系统将数据存储成块的集合。同样的方式，HDFS 以块的方式存储文件。Hadoop 应用程序负责在多个节点分配数据块。

### InputSplit

InputSplit 即我们所说的数据分片，一个单独的 mapper 处理的数据由 InputSplit 提供，即一个数据分片对应被一个 mapper 处理，数据分片会转换成记录，每个记录（即键值对）会被 map 处理。map 任务的个数等于数据分片的数量。

一开始 MapReduce 任务处理的数据是存储在输入文件的，而输入文件一般在 HDFS 。输入文件如何被读取和切分由 InputFormat 类来决定，另外它还负责创建 InputSplit。

## InputSplit 和 块的比较

让我们来讨论 MapReduce InputSplit 和块之间的特性比较。

### InputSplit 与块的大小比较

**块**：HDFS 块的默认大小是 128MB，我们可以按实际需求对该值进行配置。除了最后一个数据块，文件的其他所有块大小都是一样的，最后一个数据块大小一般小于等于 128MB。文件被切分成若干个大小为 128MB 的数据块，并存储到 Hadoop 文件系统。

**InputSplit**：默认情况下，InputSplit 的大小近似等于块大小。InputSplit 是由用户自己定义的，并且基于数据的容量可以在 MapReduce 程序调整 InputSplit 的值。

### InputSplit 和 块的数据表示

**块**：块是数据的物理表示，它包含可读或者可写的最小数据量。
**InputSplit**：它是块中数据的逻辑表示。它是在 MapReduce 程序处理数据的时候被使用的。InputSplit 本身并不存储物理数据，它只是数据的一种引用。

### InputSplit 和 块示例

![mapreduce inputsplit ](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570247580000_20191005115302653185.png)

![Mapreduce inputsplit 块](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570247625000_20191005115347185770.png)

![Mapreduce inputSplit 示例](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570247664000_20191005115426215196.png)

假如我们需要把文件存储到 HDFS。HDFS 以块的形式存储文件，块是数据读取和存储的最小单位，并且块的默认大小是 128MB 。HDFS 把文件切分成块，并把块存储在集群的不同机器节点上，假如我们有一个 130MB 的文件，那么 HDFS 会把这个文件切割成 2 个块，如上第一个图所示。

现在，如果我们想对这些块执行 MapReduce 程序，那么它是不会被处理的，因为第二个块并不是完整的块。但是这个问题 InputSplit 可以解决。InputSplit 可以把一组 block 作为一个单独的块，因为 InputSplit 里面包含下一个块的位置以及完整的块所需的数据的字节偏移量。