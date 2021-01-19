# MapReduce初识

MapReduce 是 Hadoop 生态下面的计算层，它把任务分割成小任务并分发到集群的机器上并行执行。您只需要按照 MapReduce 提供的编程接口开发业务逻辑代码即可，剩下的事情 MapReduce 框架会自动完成。比如，任务分割，任务分发等。MapReduce 程序具有函数式风格，输入是数据列表，输出依然是数据列表。MapReduce 是 Hadoop 的核心。Hadoop 如此强大就是因为 MapReduce 的并行处理能力。

## 了解 Hadoop MapReduce

MapReduce 是如何工作的呢？

MapReduce 会把打任务分成小任务，每个小任务可以在集群并行执行。每个小任务都会输出计算结果，这些结果数据后续被汇总并输出最终结果。

Hadoop MapReduce 具有较好的扩展性，它可以在很多机器上跑。集群里面单个机器可能无法执行大任务，但可以执行大任务分割后的小任务。这是 MapReduce 比较核心的机制。

## Apache MapReduce 术语

本节主要介绍 MapReduce 相关概念和术语。如，什么是 Map 和 Reduce，什么是 job，task，task attempt 等。

MapReduce 是 Hadoop 的数据处理组件。MapReduce 程序把输入数据转换成特定格式的输出数据。一个 MapReduce 程序主要就做下面这两步：

- Map
- Reduce

在 Map 和 Reduce 中间还有一个处理阶段，叫做 Shuffle 和 排序操作。

下面介绍一下 MapReduce 里面的一些关键术语。

- 什么是 MapReduce Job（作业）？

一个 MapReduce Job 过程分成两个阶段：Map 阶段和 Reduce 阶段。每个阶段都用 key/value 作为输入和输出；每个阶段都需要定义函数，也就是 map 函数和 reduce 函数；可以简单认为 map 函数是对原始数据提出出有用的部分，而 reduce 函数则是对提取出来的数据进行处理。

- 什么是 MapReduce Task

MapReduce 里面的 task 可以分两种，即 Map task 和 Reduce task，即处理分片数据的 Mapper 和 Reducer 任务，这里的 Mapper 和 Reducer 的业务逻辑由开发者定义。

- 什么是 Task Attempt

Task Attempt，即任务尝试。集群的机器在任何时间都可能发生故障，比如，正在处理数据的机器挂了，MapReduce 把任务重新调度到其他机器节点。当然这里的重新调度次数并非不受限制的，它是有上限的，默认是 4 次，如果一个任务（Mapper 任务或者 Reducer 任务）失败 4 次，那么整个 Job 就被认为失败了。对于高优先级的作业或者大型作业，这个值可以调高一点。

### Map 阶段

map 函数以键值对作为输入数据，不管数据是结构化还是非结构化，框架都会把数据转换成键值对形式。键是输入值的引用，而值就是要操作的数据集。

用户可以根据业务需求开发特定的业务逻辑来实现 MapReduce 框架的 map 函数。map 函数会对每个输入键值对的值部分处理。处理之后会生成输出结果，map 的输出叫做中间输出，它的类型可能与输入键值不同。map的输出结果是存储在本地磁盘的。

### Reduce 阶段

Reduce 以 Map 的输出结果作为输入数据，并对这些数据进行处理。通常，在 reducer 我们会做聚合或求和计算。另外，MapReduce 给 reduce 的输入数据按键做排序操作。

用户可以根据业务需求开发特定的业务逻辑来实现 MapReduce 框架的 reduce 函数，reduce 函数对输入值做聚合操作，并输出最终结果写入到 HDFS。

### Map 和 Reduce 是如何一起工作的

![mapreduce工作原理](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/file_1570082091000_20191003135451534202.png)
输入数据被分割成分片，并提供给 mapper 处理，当然，具体的 mapper 的业务逻辑需要用户来实现。所有必要的复杂的业务逻辑都在 mapper 层面实现了，繁琐的处理都由并行的 mapper 来处理了，所以，mapper 的数量要比 reducer 的数量多不少。Mapper 生成输出结果，即中间数据， 而 Reducer 以这些中间数据作为输入，具体的 Reducer 逻辑也是需要您来实现的。通常，Reducer 的逻辑相对比较简单。Reducer 执行完之后，最终结果就生成了，并且最终会存储在 HDFS。

## MapReduce 数据流程

现在让我们来看看 MapReduce 完整的数据流程，本节您将对以下问题有跟更清晰的概念：
输入数据是如何给到 mapper 的？
mapper 是如何处理数据的？
mapper 把数据结果写到哪里？
数据是如何从 mapper 流向 reducer 节点的？
reducer 在哪里执行？
reducer 将完成哪种类型的处理？

![MapReduce数据流向图](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/hadoop-mapreduce-data-flow-execution-1_20191001231203020254-20210112084021062.gif)
MapReduce 的大概的工作流程如上图所示，其中方块表示 slave 节点，所以这里有 3 个 slave 节点。 mapper 运行在 3 个 slave节点上，而 reducer 在任意一个slave 运行，上图为了简单起见，把 reducer 进程画在一个方块里，看起来是运行在一个不同的机器上，其实它是在 mapper 的节点上运行的。

Mapper 是 MapReduce 作业的第一个执行阶段。默认情况下，一个 mapper 每次处理的分片数据（split）都是一个 HDFS 数据块，mapper 的输出数据会被写到本地机器的磁盘上。mapper 跑完之后，mapper 输出的结果数据会到 reducer 节点，即运行 reducer 的机器。

Reducer 是 MapReduce 作业的 第二个执行阶段。它的计算结果将会直接落地到 HDFS。

在默认情况下，一个 slave 每次可以跑 2 个 mapper（可以根据需要调高这个值），而 slave 同时能跑多少个 mapper 取决于很多因素，比如，机器的硬件配置，HDFS 块大小等。所以建议不要把这个值调太高，因为这会降低 MapReduce 运行性能。

MapReduce 的 Mapper 会把它的输出结果写到本地磁盘。这个输出结果是临时数据，也叫做中间输出结果。所有的 mapper 都会把输出数据写到本地磁盘。mapper 执行完之后，mapper 输出的数据会从 mapper 节点传输到 reducer 节点，这个过程被称为 **shuffle**。

Reducer 也是运行在集群的任意一个 datanode 的。所有 mapper 的输出数据都会到 reducer。这些来自不同 mapper 的输出数据会被合并，并作为 reducer 的输入数据。这些合并后的数据还是存储在 mapper 所在节点的磁盘的。Reducer 是 MapReduce 框架提供的另一个你能实现自己业务逻辑的接口，通常我们会在 Reducer 做数据聚合，相加等操作。因此，Reducer 会把最终结果数据写到 HDFS。

map 和 reduce 是执行 MapReduce 作业的两个数据处理阶段。所有 mapper 执行完之后，reducer 才能开始执行。

虽然默认情况下 1 个块会存储在 3 个不同的位置，但 1 个 mapper 只处理其中的一个块。每个 mapper 的输出数据都会被传输到 reducer，因此，只有在所有的 mapper 处理完所有数据之后，reducer 才能开始处理数据。

mapper 输出数据会被 partitioner 按 key 进行分区。每个分区会基于某些条件被传输到一个 reducer。MapReduce 的工作是基于 key-value （键值对）原则的，比如，mapper 和 reducer 输入数据是键值对数据，输出同样也是键值对数据。MapReduce 数据流程是 MapReduce 框架最重要的一部分。后续章节还会详细介绍。

## MapReduce 的数据本地化

本节让我们理解一下什么是**数据本地化**，它是如何提升 MapReduce Job 的性能的。

> 移动计算比移动数据更高效

代码在离它运算的数据最近的地方执行更加高效，特别在数据量大的情况下代码执行效率提升更加明显。因为移动代码消耗的网络带宽，要远远比移动大量数据消耗的带宽资源小很多。因此，HDFS 给 MapReduce 提供了一个接口，用于把代码移动到离数据最近的地方。

因为 Hadoop 处理的数据量都比较大，经常通过网络传输大数据量并不现实，因此，它提出了一个极具创新的原则，即把计算移动到离数据最近的地方执行，而非相反，这就是我们所说的数据本地化。