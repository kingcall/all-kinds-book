# MapReduce 计数器

计数器是收集作业统计信息的有效手段之一，用于质量控制或者应用级统计。计数器还可用于辅助诊断系统故障。如果需要将日志信息传输到 map 或 reduce 任务，更好的方法通常是看能否用一个计数器值来记录某一特定事件的发生。对于大型分布式作业而言，使用计数器更加方便。除了因为获取计数器值比输出日志更方便，还有根据计数器值统计特定事件的发生次数要比分析一堆日志文件容易的多。

## 计数器分类

MapReduce 计数器可以分成两大类：

- 内置计数器
- 用户自定义计数器

下面详细讨论一下这两大类计数器：

### MapReduce 内置计数器

Hadoop 为每个作业维护若干内置计数器，以描述多项指标。例如，某些计数器记录已处理的字节数和记录数，使用户可监控已处理的输入数据量和已产生的输出数据量。

这些内置计数器被划分成若干个组。如下表：

| 组别                    | 名称/类别                                                    |
| ----------------------- | ------------------------------------------------------------ |
| MapReduce 任务计数器    | org.apache.hadoop.mapreduce.TaskCounter                      |
| 文件系统计数器          | org.apache.hadoop.mapreduce.FileSystemCounter                |
| FileInputFormat 计数器  | org.apache.hadoop.mapreduce.lib.input.FileInputFormatCounter |
| FileOutputFormat 计数器 | org.apache.hadoop.mapreduce.lib.output.FileOutputFormatCounter |
| 作业计数器              | org.apache.hadoop.mapreduce.JobCounter                       |

各组要么包含任务计数器（在任务处理过程中不断更新），要么包含作业计数器（在作业处理过程中不断更新）。

下面对各个计数器进行介绍：

#### MapReduce 任务计数器

Hadoop 任务计数器收集任务运行期间产生的信息（比如读写记录数）。比如 MAP_INPUT_RECORDS 计数器就是一个任务计数器，它记录了每个 map 任务读取记录的数量。

Hadoop 任务计数器由任务维护，它定期给 application master 发送数据。所以，它们可以全局聚合。

MapReduce 任务计数器汇总列表：

| 计数器名称                                            | 说明                                                         |
| :---------------------------------------------------- | :----------------------------------------------------------- |
| map 输入的记录数（MAP_INPUT_RECORDS）                 | 作业中所有 map 已处理的输入记录数。每次 RecorderReader 读到一条记录并将其传给 map 的 map() 函数时，该计数器的值增加。 |
| map 跳过的记录数（MAP_SKIPPED_RECORDS）               | 作业中所有 map 跳过的输入记录数。                            |
| map 输入的字节数（MAP_INPUT_BYTES）                   | 作业中所有 map 已处理的未经压缩的输入数据的字节数。每次 RecorderReader 读到一条记录并 将其传给 map 的 map() 函数时，该计数器的值增加 |
| 分片split的原始字节数（SPLIT_RAW_BYTES）              | 由 map 读取的输入-分片对象的字节数。这些对象描述分片元数据（文件的位移和长度），而不是分片的数据自身，因此总规模是小的 |
| map 输出的记录数（MAP_OUTPUT_RECORDS）                | 作业中所有 map 产生的 map 输出记录数。每次某一个 map 的Context 调用 write() 方法时，该计数器的值增加 |
| map 输出的字节数（MAP_OUTPUT_BYTES）                  | 作业中所有 map 产生的 未经压缩的输出数据的字节数。每次某一个 map 的 Context 调用 write() 方法时，该计数器的值增加。 |
| map 输出的物化字节数（MAP_OUTPUT_MATERIALIZED_BYTES） | map 输出后确实写到磁盘上的字节数；若 map 输出压缩功能被启用，则会在计数器值上反映出来 |
| combine 输入的记录数（COMBINE_INPUT_RECORDS）         | 作业中所有 Combiner（如果有）已处理的输入记录数。Combiner 的迭代器每次读一个值，该计数器的值增加。 |
| combine 输出的记录数（COMBINE_OUTPUT_RECORDS）        | 作业中所有 Combiner（如果有）已产生的输出记录数。每当一个 Combiner 的 Context 调用 write() 方法时，该计数器的值增加。 |
| reduce 输入的组（REDUCE_INPUT_GROUPS）                | 作业中所有 reducer 已经处理的不同的码分组的个数。每当某一个 reducer 的 reduce() 被调用时，该计数器的值增加。 |
| reduce 输入的记录数（REDUCE_INPUT_RECORDS）           | 作业中所有 reducer 已经处理的输入记录的个数。每当某个 reducer 的迭代器读一个值时，该计数器的值增加。如果所有 reducer 已经处理完所有输入， 则该计数器的值与计数器 “map 输出的记录” 的值相同 |
| reduce 输出的记录数（REDUCE_OUTPUT_RECORDS）          | 作业中所有 map 已经产生的 reduce 输出记录数。每当某一个 reducer 的 Context 调用 write() 方法时，该计数器的值增加。 |
| reduce 跳过的组数（REDUCE_SKIPPED_GROUPS）            | 作业中所有 reducer 已经跳过的不同的码分组的个数。            |
| reduce 跳过的记录数（REDUCE_SKIPPED_RECORDS）         | 作业中所有 reducer 已经跳过输入记录数。                      |
| reduce 经过 shuffle 的字节数（REDUCE_SHUFFLE_BYTES）  | shuffle 将 map 的输出数据复制到 reducer 中的字节数。         |
| 溢出的记录数（SPILLED_RECORDS）                       | 作业中所有 map和reduce 任务溢出到磁盘的记录数                |
| CPU 毫秒（CPU_MILLISECONDS）                          | 总计的 CPU 时间，以毫秒为单位，由/proc/cpuinfo获取           |
| 物理内存字节数（PHYSICAL_MEMORY_BYTES）               | 一个任务所用物理内存的字节数，由/proc/cpuinfo获取            |
| 虚拟内存字节数（VIRTUAL_MEMORY_BYTES）                | 一个任务所用虚拟内存的字节数，由/proc/cpuinfo获取            |
| 有效的堆字节数（COMMITTED_HEAP_BYTES）                | 在 JVM 中的总有效内存量（以字节为单位），可由Runtime().getRuntime().totaoMemory()获取。 |
| GC 运行时间毫秒数（GC_TIME_MILLIS）                   | 在任务执行过程中，垃圾收集器（garbage collection）花费的时间（以毫秒为单位）， 可由 GarbageCollector MXBean.getCollectionTime()获取；该计数器并未出现在1.x版本中。 |
| 由 shuffle 传输的 map 输出数（SHUFFLED_MAPS）         | 有 shuffle 传输到 reducer 的 map 输出文件数。                |
| 失败的 shuffle 数（SHUFFLE_MAPS）                     | 在 shuffle 过程中，发生拷贝错误的 map 输出文件数，该计数器并没有包含在 1.x 版本中。 |
| 被合并的 map 输出数（MERGED_MAP_OUTPUTS）             | 在 shuffle 过程中，在 reduce 端被合并的 map 输出文件数，该计数器没有包含在 1.x 版本中。 |

#### 文件系统计数器

Hadoop 文件系统计数器会收集从文件系统读写字节数相关的信息。下面是文件系统计数器的名称以及功能说明：

文件系统的写字节数（BYTES_WRITTEN） —— map 或者 reduce 任务写入文件系统的字节数
文件系统的读字节数（BYTES_READ） —— map 或者 reduce 任务从文件系统读取的字节数

#### FileInputFormat 计数器

读取的字节数（BYTES_READ）—— 由 map 任务通过 FileInputFormat 读取的字节数。

#### FileOutputformat 计数器

写的字节数（BYTES_WRITTEN）—— 由 map 任务（针对仅 map 的作业）或者 reduce 任务通过 FileOutputformat 写的字节数。

#### MapReduce 作业计数器

作业计数器由 application master 维护，因此无需通过网络传输数据，这一点包括 “用户定义的计数器” 在内的其他计数器不同。这些计数器都是作业级别的统计，其值不会随着任务运行而改变。例如，TOTAL_LAUNCHED_MAPS 统计的作业执行过程中的启动的 map 任务数，包括失败的 map 任务。

作业计数器汇总列表：

| 计数器名称                                                   | 说明                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| 启用的map任务数（TOTAL_LAUNCHED_MAPS）                       | 启动的map任务数，包括以“推测执行” 方式启动的任务。           |
| 启用的 reduce 任务数（TOTAL_LAUNCHED_REDUCES）               | 启动的reduce任务数，包括以“推测执行” 方式启动的任务。        |
| 失败的map任务数（NUM_FAILED_MAPS）                           | 失败的map任务数。                                            |
| 失败的 reduce 任务数（NUM_FAILED_REDUCES）                   | 失败的reduce任务数。                                         |
| 数据本地化的 map 任务数（DATA_LOCAL_MAPS）                   | 与输入数据在同一节点的 map 任务数。                          |
| 机架本地化的 map 任务数（RACK_LOCAL_MAPS）                   | 与输入数据在同一机架范围内、但不在同一节点上的 map 任务数。  |
| 其它本地化的 map 任务数（OTHER_LOCAL_MAPS）                  | 与输入数据不在同一机架范围内的 map 任务数。由于机架之间的宽带资源相对较少，Hadoop 会尽量让 map 任务靠近输入数据执行，因此该计数器值一般比较小。 |
| map 任务的总运行时间（SLOTS_MILLIS_MAPS）                    | map 任务的总运行时间，单位毫秒。该计数器包括以推测执行方式启动的任务。 |
| reduce 任务的总运行时间（SLOTS_MILLIS_REDUCES）              | reduce任务的总运行时间，单位毫秒。该值包括以推测执行方式启动的任务。 |
| 在保留槽之后，map任务等待的总时间（FALLOW_SLOTS_MILLIS_MAPS） | 在为 map 任务保留槽之后所花费的总等待时间，单位是毫秒。      |
| 在保留槽之后，reduce 任务等待的总时间（FALLOW_SLOTS_MILLIS_REDUCES） | 在为 reduce 任务保留槽之后，花在等待上的总时间，单位为毫秒。 |

### 用户自定义计数器

MapReduce 允许用户编写程序来自定义计数器，计数器的值可在 mapper 或者 reducer 中增加，计数器由一个 Java 枚举（enum）类型来定义，以便对有关的计数器分组。一个作业可以定义的枚举类型数量不限，各个枚举类型所包含的字段数量也不限。枚举类型的名称即为组的名称，枚举类型的字段就是计数器名称。计数器是全局的。换句话说，MapReduce 框架将跨所有 map 和 reduce 聚集这些计数器。并在作业结束时产生一个最终结果。

#### 动态计数器

由于 Java 枚举类型的字段是在编译阶段就必须指定的，因而无法使用枚举类型动态新建计数器。为了解决这个问题，我们可以使用动态计数器，它是一种不由 Java 枚举类型定义的计数器。