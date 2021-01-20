# MapReduce Combiner

当我们用 MapReduce 作业处理大数据集的时候，Mapper 生成的中间结果数据就会比较大，而且这些中间结果数据后续会被传输到 Reducer 继续处理，这会导致非常大的网络开销，甚至网络堵塞。MapReduce 框架提供了一个函数——**Hadoop Combiner 函数**，它在减少网络阻塞方面扮演着一个关键的角色。

我们在之前已经学习了 Hadoop MapReduce 框架的 mapper 和 reducer。现在我们来学习 Hadoop MapReduce 框架的 Combiner。

MapReduce combiner 也被称为 “微型 reducer ”。combiner 的主要工作就是在 Mapper 的输出数据被传输到 Reducer 之前对这些数据进行处理。它在 mapper 之后 reducer 之前执行，也就是在 mapper 和 reducer 两个阶段的中间执行。并且 combiner 的执行是可选的，即可用可不用。

## MapReduce combiner 工作原理

让我们来理解一下 Hadoop combiner 的工作原理，以及比较一下使用了 combiner 和未使用两者的区别。

### 未使用 combiner 的 MapReduce 程序

![未使用combiner的mapreduce程序](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570245882000_20191005112444881378-20210112084811061.png)

在上图中，MapReduce 程序没有使用 combiner。输入数据被分到两个 mapper 里面，并且生成了 9 个 key。现在 mapper 的中间结果已经产生了，即上图所示的 9 个键值对。后续 mapper 将会把这些数据直接发送到 reducer。在数据发送给 reducer 期间，这些数据的传输会消耗一些网络带宽（带宽，即在两台机器间传输数据的时间消耗）。如果数据容量很大，那么数据传输到 reducer 的耗时就更长。

如果我们在 mapper 和 reducer 中间使用了 combiner，那么，数据在从 mapper 传输到 reducer 之前，combiner 会对数据按 key 做聚合，那么输出的数据就是 4 个键值对。

### 使用 combiner 的 MapReduce 程序

![使用combiner的mapreduce程序](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570245955000_20191005112557054455-20210112084843341.png)

在使用了 combiner 之后，Reducer 只需要处理由 2 个 combiner 生成的 4 个键值对数据。因此，reducer 生成最终结果只需要执行 4 次即可。作业的整体性能得到显著提升。

## MapReduce combiner 优点

我们已经详细讨论了什么是 Hadoop MapReduce combiner，现在我们讨论一下使用 MapReduce combiner 的优势。

- Hadoop combiner 减少了数据从 mapper 到 reducer 之间的传输时间。
- 减少 reducer 处理的数据量，但最终计算结果不变。
- 提升 reducer 的整体性能。

## MapReduce combiner 缺点

- MapReduce job 不能依赖于 Hadoop combiner 的执行，因为 combiner 的执行没有保证。
- 在本地文件系统中，键值对是存储在 Hadoop 中的，运行 combiner 会导致昂贵的磁盘IO 开销。



## 使用MapReduce combiner 优化程序

### 利用Combiner计算每一年的平均气温

