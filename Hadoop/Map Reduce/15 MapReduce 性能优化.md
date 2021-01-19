# MapReduce 性能优化

对 MapReduce 作业进行性能调优，需要从 MapReduce 的原理出发。下面来重温一下 MapReduce 原理，并对各个阶段进行做相应优化。

![mapreduce性能优化](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570260094000_20191005152135801998.png)

## Map阶段

### 读数据从HDFS读取数据

#### 读取数据产生多少个 Mapper？

Mapper 数据过大的话，会产生大量的小文件，由于 Mapper 是基于虚拟机的，过多的 Mapper 创建和初始化及关闭虚拟机都会消耗大量的硬件资源。

#### Mapper 数量由什么决定？

Mapper 的数量由下面三个因素决定：
（1）输入文件数目
（2）输入文件的大小
（3）配置参数

其中涉及的参数：
启动 map 最小的 split size 大小，默认 0
`mapreduce.input.fileinputformat.split.minsize`

启动 map 最大的 split size 大小，默认 256MB
`mapreduce.input.fileinputformat.split.maxsize`

HDFS 块大小，默认 128MB
`dfs.block.size`

数据分片计算公式如下：
`splitSize = Math.max(minSize, Math.min(maxSize, blockSize));`

默认情况下，假如一个文件 800M，Block 大小是 128M，那么 Mapper 数目就是 7个。6 个 Mapper 处理的数据是 128M，1个 Mapper 处理的数据是 32M。

假如一个目录下有三个文件大小分别为：5M、10M、150M，这时其实会产生 4 个Mapper，处理的数据分别是 5M，10M，128M，22M。

Mapper 是基于文件自动产生的，如果想要自己控制 Mapper 的个数该怎么做？就如上面的第二个例子。5M，10M 的数据很快处理完了，128M 要很长时间。这个就需要通过参数的控制来调节 Mapper 的个数。减少 Mapper 的个数的话，就要合并小文件，这种小文件有可能是直接来自于数据源的小文件，也可能是 Reduce 产生的小文件。

设置 combiner：（set都是在hive脚本，也可以配置Hadoop）
设置合并器本身：

```
set hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat;set hive.merge.mapFiles=true;set hive.merge.mapredFiles=true;set hive.merge.size.per.task=256000000; //每个 Mapper 要处理的数据，就把上面的5M，10M……合并成为一个
```

一般还要配合下面的参数：

```
set mapred.max.split.size=256000000 // mapred切分的大小set mapred.min.split.size.per.node=128000000  //低于128M就算小文件，数据在一个节点会合并，在多个不同的节点会把数据抓过来进行合并。
```

Hadoop中的参数：
可以通过控制文件的数量控制mapper数量

```
mapreduce.input.fileinputformat.split.minsize  //默认值为 1，小于这个值会合并。mapreduce.input.fileinputformat.split.maxsize //大于这个值会切分
```

### 处理数据

#### Partition 说明

对于 map 输出的每一个键值对，系统都会给定一个 partition，partition值默认是通过计算 key 的 hash 值后对 Reduce task 的数量取模获得。如果一个键值对的 partition 值为 1，意味着这个键值对会交给第一个 Reducer 处理。

#### 自定义 partitioner 的情况

- 我们知道每一个 Reduce 的输出都是有序的，但是将所有 Reduce 的输出合并到一起却并非是全局有序的，如果要做到全局有序，我们该怎么做呢？最简单的方式，只设置一个 Reduce task，但是这样完全发挥不出集群的优势，而且能应对的数据量也很受限。最佳的方式是自己定义一个 Partitioner，用输入数据的最大值除以系统Reduce task 数量的商作为分割边界，也就是说分割数据的边界为此商的1倍、2倍至numPartitions-1倍，这样就能保证执行 partition 后的数据是整体有序的。
- 解决数据倾斜，另一种需要我们自己定义一个Partitioner的情况是各个 Reduce task 处理的键值对数量极不平衡。对于某些数据集，由于很多不同的 key 的 hash 值都一样，导致这些键值对都被分给同一个 Reducer 处理，而其他的 Reducer 处理的键值对很少，从而拖延整个任务的进度。当然，编写自己的 Partitioner 必须要保证具有相同key值的键值对分发到同一个 Reducer。
- 自定义的 Key 包含了好几个字段，比如自定义 key 是一个对象，包括 type1，type2，type3，只需要根据 type1 去分发数据，其他字段用作二次排序。

#### 环形缓冲区

Map 的输出结果是由 collector 处理的，每个 Map 任务不断地将键值对输出到在内存中构造的一个环形数据结构中。使用环形数据结构是为了更有效地使用内存空间，在内存中放置尽可能多的数据。

这个数据结构其实就是个字节数组，叫 Kvbuffer，名如其义，但是这里面不光放置了数据，还放置了一些索引数据，给放置索引数据的区域起了一个 Kvmeta 的别名，在Kvbuffer 的一块区域上穿了一个 IntBuffer（字节序采用的是平台自身的字节序）的马甲。数据区域和索引数据区域在 Kvbuffer 中是相邻不重叠的两个区域，用一个分界点来划分两者，分界点不是亘古不变的，而是每次 Spill 之后都会更新一次。初始的分界点是0，数据的存储方向是向上增长，索引数据的存储方向是向下增长的，Kvbuffer存放指针的 bufindex 是一直向上增长，比如 bufindex 初始值为0，一个 Int 型的 key 写完之后，bufindex 增长为4，一个 Int 型的 value 写完之后，bufindex 增长为8。

索引是对在 kvbuffer 中的键值对的索引，是个四元组，包括：value 的起始位置、key 的起始位置、partition 值、value 的长度，占用四个 Int 长度，Kvmeta 的存放指针的Kvindex 每次都是向下跳四个“格子”，然后再向上一个格子一个格子地填充四元组的数据。比如 Kvindex 初始位置是 -4，当第一个键值对写完之后，(Kvindex+0) 的位置存放 value 的起始位置、(Kvindex+1) 的位置存放 key 的起始位置、(Kvindex+2) 的位置存放 partition 的值、(Kvindex+3) 的位置存放 value 的长度，然后 Kvindex 跳到 -8 的位置，等第二个键值对和索引写完之后，Kvindex 跳到 -12 位置。

### 写数据到磁盘

Mapper 中的 Kvbuffer 的大小默认100M，可以通过`mapreduce.task.io.sort.mb（默认值：100）`参数来调整。可以根据不同的硬件尤其是内存的大小来调整，调大的话，会减少磁盘 spill 的次数，此时如果内存足够的话，一般都会显著提升性能。spill 一般会在 Buffer 空间大小的 80% 开始进行 spill（因为spill的时候还有可能别的线程在往里写数据，因为还预留空间，有可能有正在写到 Buffer 中的数据），可以通过 `mapreduce.map.sort.spill.percent（默认值：0.80）`进行调整，Map Task 在计算的时候会不断产生很多 spill 文件，在 Map Task 结束前会对这些 spill 文件进行合并，这个过程就是 merge 的过程。`mapreduce.task.io.sort.factor（默认值：10）`，代表进行 merge 的时候最多能同时 merge 多少个 spill，如果有 100 个 spill 文件，此时就无法一次完成整个 merge 的过程，这个时候需要调大 `mapreduce.task.io.sort.factor（默认值：10）`来减少 merge 的次数，从而减少磁盘的操作；

Spill 这个重要的过程是由 Spill 线程承担，Spill 线程从 Map 任务接到指令之后就开始正式干活，干的活叫 SortAndSpill，原来不仅仅是 Spill，在 Spill 之前还有个颇具争议性的 Sort。

Combiner 存在的时候，此时会根据 Combiner 定义的函数对 map 的结果进行合并，什么时候进行 Combiner 操作呢？和 Map 在一个 JVM 中，是由 `min.num.spill.for.combine` 的参数决定的，默认是3，也就是说 spill 的文件数在默认情况下有三个的时候就要进行 combine 操作，最终减少磁盘数据。

减少磁盘 IO 和网络 IO 还可以通过压缩，对 spill，merge 文件都可以进行压缩。中间结果非常的大，IO 成为瓶颈的时候压缩就非常有用，可以通过`mapreduce.map.output.compress（默认值：false）`设置为 true 进行压缩，数据会被压缩写入磁盘，读数据读的是压缩数据需要解压，在实际经验中 Hive 在 Hadoop 的运行的瓶颈一般都是 IO 而不是 CPU，压缩一般可以 10 倍的减少 IO 操作，压缩的方式 Gzip，Lzo，BZip2，Lzma 等，其中 Lzo 是一种比较平衡选择，`mapreduce.map.output.compress.codec（默认：org.apache.hadoop.io.compress.DefaultCodec）`参数设置。但这个过程会消耗 CPU，适合 IO 瓶颈比较大。

## Shuffle 和 Reduce 阶段

### Copy

由于 job 的每一个 map 都会根据 `reduce(n)` 数将数据分成 map 输出结果分成 n 个 partition，所以 map 的中间结果中是有可能包含每一个 reduce 需要处理的部分数据的。所以，为了优化reduce 的执行时间，hadoop 中是等 job 的第一个 map 结束后，所有的 reduce 就开始尝试从完成的 map 中下载该 reduce 对应的 partition 部分数据，因此 map 和 reduce 是交叉进行的，其实就是 shuffle。Reduce 任务通过 HTTP 向各个 Map 任务拖取（下载）它所需要的数据（网络传输），Reducer 是如何知道要去哪些机器取数据呢？一旦 map 任务完成之后，就会通过常规心跳通知应用程序的 Application Master。reduce 的一个线程会周期性地向 master 询问，直到提取完所有数据，数据被 reduce 提走之后，map 机器不会立刻删除数据，这是为了预防 reduce 任务失败需要重做。因此 map 输出数据是在整个作业完成之后才被删除掉的。

reduce 进程启动数据 copy 线程(Fetcher)，通过 HTTP 方式请求 maptask 所在的 TaskTracker 获取 maptask 的输出文件。由于 map 通常有许多个，所以对一个 reduce 来说，下载也可以是并行的从多个 map 下载，那到底同时到多少个 Mapper 下载数据？这个并行度是可以通过`mapreduce.reduce.shuffle.parallelcopies(默认值：5）`调整。默认情况下，每个 Reducer 只会有5 个 map 端并行的下载线程从map下数据，如果一个时间段内 job 完成的 map 有 100 个或者更多，那么 reduce 也最多只能同时下载 5 个 map 的数据，所以这个参数比较适合 map 很多并且完成的比较快的 job 的情况下调大，有利于 reduce 更快的获取属于自己部分的数据。 在Reducer 内存和网络都比较好的情况下，可以调大该参数。

reduce 的每一个下载线程在下载某个 map 数据的时候，有可能因为那个 map 中间结果所在机器发生错误，或者中间结果的文件丢失，或者网络瞬断等等情况，这样 reduce 的下载就有可能失败，所以 reduce 的下载线程并不会无休止的等待下去，当一定时间后下载仍然失败，那么下载线程就会放弃这次下载，并在随后尝试从另外的地方下载（因为这段时间 map 可能重跑）。reduce 下载线程的这个最大的下载时间段是可以通过`mapreduce.reduce.shuffle.read.timeout（默认值：180000秒）`调整的。如果集群环境的网络本身是瓶颈，那么用户可以通过调大这个参数来避免 reduce 下载线程被误判为失败的情况。一般情况下都会调大这个参数，这是企业级最佳实战。

### MergeSort

这里的 merge 和 map 端的 merge 动作类似，只是数组中存放的是不同 map 端 copy 来的数值。Copy 过来的数据会先放入内存缓冲区中，然后当使用内存达到一定量的时候才 spill 磁盘。这里的缓冲区大小要比 map 端的更为灵活，它基于 JVM 的 heap size 设置。这个内存大小的控制就不像 map 一样可以通过 `io.sort.mb` 来设定了，而是通过另外一个参数 `mapreduce.reduce.shuffle.input.buffer.percent（默认值：0.7）` 来设置，这个参数其实是一个百分比，意思是说，shuffile 在 reduce 内存中的数据最多使用内存量为：0.7 × maxHeap of reduce task。JVM 的 heapsize 的 70%。内存到磁盘 merge 的启动门限可以通过`mapreduce.reduce.shuffle.merge.percent（默认值：0.66）`配置。也就是说，如果该 reduce task 的最大 heap 使用量（通常通过 `mapreduce.admin.reduce.child.java.opts` 来设置，比如设置为 `-Xmx1024m` ）的一定比例用来缓存数据。默认情况下，reduce 会使用其 heapsize 的 70%来在内存中缓存数据。假设 `mapreduce.reduce.shuffle.input.buffer.percent` 为0.7，reducetask 的 max heapsize为1G，那么用来做下载数据缓存的内存就为大概 700MB 左右。这 700M 的内存，跟 map 端一样，也不是要等到全部写满才会往磁盘刷的，而是当这 700M 中被使用到了一定的限度（通常是一个百分比），就会开始往磁盘刷（刷磁盘前会先做 sortMerge ）。这个限度阈值也是可以通过参数 `mapreduce.reduce.shuffle.merge.percent（默认值：0.66）`来设定。与map 端类似，这也是溢写的过程，这个过程中如果你设置有 Combiner，也是会启用的，然后在磁盘中生成了众多的溢写文件。这种 merge 方式一直在运行，直到没有 map 端的数据时才结束，然后启动磁盘到磁盘的 merge 方式生成最终的那个文件。

这里需要强调的是，merge 有三种形式：

- 内存到内存（memToMemMerger）
- 内存中 Merge（inMemoryMerger）
- 磁盘上的 Merge（onDiskMerger）具体包括两个：Copy过程中磁盘合并和磁盘到磁盘。

#### 内存到内存Merge（memToMemMerger）

Hadoop 定义了一种 MemToMem 合并，这种合并将内存中的 map 输出合并，然后再写入内存。这种合并默认关闭，可以通过 `mapreduce.reduce.merge.memtomem.enabled (默认值：false)` 打开，当 map 输出文件达到 mapreduce.reduce.merge.memtomem.threshold 时，触发这种合并。

#### 内存中Merge（inMemoryMerger）

当缓冲中数据达到配置的阈值时，这些数据在内存中被合并、写入机器磁盘。

阈值有2种配置方式：

- **配置内存比例**：前面提到 reduceJVM 堆内存的一部分用于存放来自 map 任务的输入，在这基础之上配置一个开始合并数据的比例。假设用于存放 map 输出的内存为 500M， mapreduce.reduce.shuffle.merge.percent 配置为0.66，则当内存中的数据达到 330M 的时候，会触发合并写入。
- **配置 map 输出数量**： 通过 mapreduce.reduce.merge.inmem.threshold 配置。在合并的过程中，会对被合并的文件做全局的排序。如果作业配置了 Combiner，则会运行 combine 函数，减少写入磁盘的数据量。

#### 磁盘上的Merge（onDiskMerger）

- Copy 过程中磁盘 Merge：在 copy 过来的数据不断写入磁盘的过程中，一个后台线程会把这些文件合并为更大的、有序的文件。如果 map 的输出结果进行了压缩，则在合并过程中，需要在内存中解压后才能给进行合并。这里的合并只是为了减少最终合并的工作量，也就是在 map 输出还在拷贝时，就开始进行一部分合并工作。合并的过程一样会进行全局排序。
- 最终磁盘中 Merge：当所有 map 输出都拷贝完毕之后，所有数据被最后合并成一个整体有序的文件，作为 reduce 任务的输入。这个合并过程是一轮一轮进行的，最后一轮的合并结果直接推送给reduce 作为输入，节省了磁盘操作的一个来回。最后（所有 map 输出都拷贝到 reduce 之后）进行合并的 map 输出可能来自合并后写入磁盘的文件，也可能来及内存缓冲，在最后写入内存的 map 输出可能没有达到阈值触发合并，所以还留在内存中。

每一轮合并不一定合并平均数量的文件数，指导原则是使用整个合并过程中写入磁盘的数据量最小，为了达到这个目的，则需要最终的一轮合并中合并尽可能多的数据，因为最后一轮的数据直接作为 reduce 的输入，无需写入磁盘再读出。因此我们让最终的一轮合并的文件数达到最大，即合并因子的值，通过 `mapreduce.task.io.sort.factor（默认值：10）`来配置。

如本文开头的图所示，Reduce 阶段中一个 Reduce 过程可能的合并方式为：假设现在有20个 map 输出文件，合并因子配置为5，则需要4轮的合并。最终的一轮确保合并5个文件，其中包括2个来自前2轮的合并结果，因此原始的20个中，再留出3个给最终一轮。

### Reduce函数调用（用户自定义业务逻辑）

当 reduce 将所有的 map 上对应自己 partition 的数据下载完成后，就会开始真正的 reduce 计算阶段。reduce task 真正进入 reduce 函数的计算阶段，由于 reduce 计算时肯定也是需要消耗内存的，而在读取reduce 需要的数据时，同样是需要内存作为 buffer，这个参数是控制，reducer 需要多少的内存百分比来作为 reduce 读已经 sort 好的数据的 buffer 大小？默认用多大内存呢？默认情况下为 0，也就是说，默认情况下，reduce 是全部从磁盘开始读处理数据。可以用 `mapreduce.reduce.input.buffer.percent（默认值0.0）`(源代码 MergeManagerImpl.java：674行)来设置 reduce 的缓存。如果这个参数大于0，那么就会有一定量的数据被缓存在内存并输送给 reduce，当 reduce 计算逻辑消耗内存很小时，可以分一部分内存用来缓存数据，可以提升计算的速度。所以默认情况下都是从磁盘读取数据，如果内存足够大的话，务必设置该参数让 reduce 直接从缓存读数据，这样做就有点 Spark Cache 的感觉。

Reduce 在这个阶段，框架为已分组的输入数据中的每个 `<key, (list of values)>`对调用一次 `reduce(WritableComparable,Iterator, OutputCollector, Reporter)`方法。Reduce任务的输出通常是通过调用 `OutputCollector.collect(WritableComparable,Writable)` 写入文件系统的。Reducer 的输出是没有排序的。

### 性能调优

如果能够根据情况对 shuffle 过程进行调优，对于提供 MapReduce 性能很有帮助。相关的参数配置列在后面的表格中。

一个通用的原则是给 shuffle 过程分配尽可能大的内存，当然你需要确保 map 和 reduce 有足够的内存来运行业务逻辑。因此在实现 Mapper 和 Reducer 时，应该尽量减少内存的使用，例如避免在 Map 中不断地叠加。

运行 map 和 reduce 任务的 JVM，内存通过 `mapred.child.java.opts` 属性来设置，尽可能设大内存。容器的内存大小通过 `mapreduce.map.memory.mb` 和 `mapreduce.reduce.memory.mb` 来设置，默认都是 1024M。

#### map优化

在 map 端，避免写入多个 spill 文件可能达到最好的性能，一个 spill 文件是最好的。通过估计 map 的输出大小，设置合理的 mapreduce.task.io.sort.* 属性，使得 spill 文件数量最小。例如尽可能调大 `mapreduce.task.io.sort.mb`。

map 端相关的属性如下表：
![map端性能优化](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570260954000_20191005153556539868.png)

#### reduce 优化

在 reduce 端，如果能够让所有数据都保存在内存中，可以达到最佳的性能。通常情况下，内存都保留给 reduce 函数，但是如果 reduce 函数对内存需求不是很高，将 `mapreduce.reduce.merge.inmem.threshold`（触发合并的 map 输出文件数）设为 0，`mapreduce.reduce.input.buffer.percent`（用于保存 map 输出文件的堆内存比例）设为1.0，可以达到很好的性能提升。在2008年的TB级别数据排序性能测试中，Hadoop 就是通过将 reduce 的中间数据都保存在内存中胜利的。

reduce端相关属性：
![reduce端优化](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570261039000_20191005153721029847.png)

#### 通用优化

Hadoop 默认使用 4KB 作为缓冲，这个算是很小的，可以通过 `io.file.buffer.size` 来调高缓冲池大小。