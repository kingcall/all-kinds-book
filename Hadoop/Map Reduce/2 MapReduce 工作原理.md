# MapReduce 工作原理

[TOC]



本节主要详细介绍 MapReduce 的工作原理。

## 核心组件


![mapreduce 工作原理](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/clipboard_20191004215521283279.png)

上面这个流程图已经把 MapReduce 的工作过程说的很清楚了，下面我们来一个一个拆解一下。

### 输入文件

首先，MapReduce 任务的目的是处理数据，那数据从哪里来？一般一个 MapReduce 任务的输入数据是来自于 HDFS 文件，这里的数据文件就叫做 MapReduce 任务的输入文件，而 HDFS 上文件的格式多种多样，比如有文本文件，二进制文件等。

### InputFormat

InputFormat 是 MapReduce 框架的一个类，它对输入文件进行分割和读取，并创建数据分片 InputSplit。

### InputSplit

InputSplit 对象即数据分片对象，由 InputFormat 生成的，一个数据分片由一个 Mapper 来处理，数据分片是逻辑上的划分，并非物理分割。每一个分片都会相应创建一个 map 任务，因此，map 任务的数量等于分片的数量，即有多少个分片就有多少个 map 任务。分片会被划分成记录，并且每个记录都会被对应 mapper 处理。

### RecordReader

它会跟 InputSplit 交互，并把数据转换成适合 mapper 读取的键值对（key-value pair）记录。默认情况下，它用的是 TextInputFormat 类来做转换。RecordReader 与 InputSplit 交互一直到文件读取完成。它会给文件的每一行数据分配一个字节偏移量（byte offset）作为唯一编号。后续这些键值对将被发送给 mapper 做进一步处理。

### Mapper

它负责处理每一个来自 RecordReader 的记录，并生成新的键值对数据，这些 Mapper 新生成的键值对跟输入键值对是不一样的。Mapper 的输出也就是我们前面说的中间结果将会被写到本地磁盘。Mapper 的输出数据并不是存储在 HDFS 的，因为这是临时数据，如果把临时数据写到 HDFS ，将造成不必要的复制，会导致 map 任务性能低下。Mapper 的输出数据被传输给 Combiner 做下一步处理。

### Combiner

combiner 其实是一种 reduce 操作。它会对 mapper 的输出数据做本地聚合，也就是说它是在输出数据的 mapper 所在的机器上执行的。主要为了减少 mapper 和 reducer 之间的数据传输。combiner 执行完成之后，它的输出结果就会被传输到 partitioner 做下一步处理。

### Partitioner

如果一个 MapReduce 作业在 reduce 阶段有多个 reducer 任务参与，才会有 Partitioner 这一步，即数据分区。如果只有一个 reducer 任务，Partitioner 是不会执行的，即不会对数据分区。

Partitioner 对来自 combiner 的输出数据分区并排序，其实就是对数据的 key 做哈希运算，具有相同 key 的记录会被分到相同的分区，然后每个分区会被发送给 reducer。

### Shuffle 和排序

现在，Partitioner 的输出被 shuffle 到 reduce 节点（ 这里的 reduce 节点其实就是正常的 slave 节点，由于在上面跑 reduce 任务所以才叫 reduce 节点）。shuffle 是对数据进行跨网络的物理移动，需要消耗网络带宽资源。在所有 mapper 都完成之后，他们的输出数据才会被 shuffle 到 reduce 节点，并且这些 mapper 产生的数据会被合并和排序，然后作为 reduce 阶段的输入数据。

### Reducer

在 reduce 阶段，它把 mapper 输出的键值对数据作为输入，然后对每个键值对数据记录应用 reducer 函数并输出结果。reducer 的输出数据是 MapReduce 作业的最终计算结果，它会被存储到 HDFS。

### RecordWrite

它负责把来自 Reducer 输出的键值对数据写到输出文件。

### OutputFormat

RecordWriter 将 Reducer 输出的键值对写入输出文件的方式由 OutputFormat 决定。OutputFormat 是由 Hadoop 提供的用于把数据写到 HDFS 或者本地磁盘的接口。因此，reducer 的最终输出数据是由 Outputformat 实例负责写入到 HDFS 的。

以上就是 MapReduce 完整的工作流程了。后续的教程会对每个步骤进行详细分析。



## 作业的执行流程

　　一个mapreduce作业的执行流程是：作业提交->作业初始化->任务分配->任务执行->更新任务执行进度和状态->作业完成。 

　　![img](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/999804-20171025164459238-1429314889.png)

　　一个完整的mapreduce作业流程，包括4个独立的实体：
　　　　客户端：client，编写mapreduce程序，配置作业，提交作业。
　　　　JobTracker：协调这个作业的运行，分配作业，初始化作业，与TaskTracker进行通信。
　　　　TaskTracker：负责运行作业，保持与JobTracker进行通信。
　　　　HDFS：分布式文件系统，保持作业的数据和结果。

### 1、提交作业

　　JobClient使用runjob方法创建一个JobClient实例，然后调用submitJob()方法进行作业的提交，提交作业的具体过程如下：

　　　　1）通过调用JobTracker对象的getNewJobId()方法从JobTracker处获得一个作业ID。
　　　　2）检查作业的相关路径。如果输出路径存在，作业将不会被提交（保护上一个作业运行结果）。
　　　　3）计算作业的输入分片，如果无法计算，例如输入路径不存在，作业将不被提交，错误返回给mapreduce程序。
　　　　4）将运行作业所需资源(作业jar文件，配置文件和计算得到的分片)复制到HDFS上。
　　　　5）告知JobTracker作业准备执行（使用JobTracker对象的submitJob()方法来真正提交作业）。

### 2、作业初始化

　　当JobTracker收到Job提交的请求后，将Job保存在一个内部队列，并让Job Scheduler（作业调度器）处理并初始化。初始化涉及到创建一个封装了其tasks的job对象，

　　并保持对task的状态和进度的跟踪(step 5)。当创建要运行的一系列task对象后，Job Scheduler首先开始从文件系统中获取由JobClient计算的input splits(step 6)，然后

　　再为每个split创建map task。

### 3、任务的分配

　　TaskTracker和JobTracker之间的通信和任务分配是通过心跳机制完成的。TaskTracker作为一个单独的JVM，它执行一个简单的循环，主要实现每隔一段时间向JobTracker

　　发送心跳，告诉JobTracker此TaskTracker是否存活，是否准备执行新的任务。如果有待分配的任务，它就会为TaskTracker分配一个任务。

### 4、任务的执行

　　TaskTracker申请到新的任务之后，就要在本地运行了。首先，是将任务本地化（包括运行任务所需的数据、配置信息、代码等），即从HDFS复制到本地。调用localizeJob()完成的。

　　对于使用Streaming和Pipes创建Map或者Reduce程序的任务，Java会把key/value传递给外部进程，然后通过用户自定义的Map或者Reduce进行处理，然后把key/value传回到java中。

　　其中就好像是TaskTracker的子进程在处理Map和Reduce代码一样。

### 5、更新任务的执行进度和状态

　　进度和状态是通过heartbeat(心跳机制)来更新和维护的。对于Map Task，进度就是已处理数据和所有输入数据的比例。对于Reduce Task，情况就邮电复杂，包括3部分，

　　拷贝中间结果文件、排序、reduce调用，每部分占1/3。

### 6、任务完成

　　当Job完成后，JobTracker会收一个Job Complete的通知，并将当前的Job状态更新为successful，同时JobClient也会轮循获知提交的Job已经完成，将信息显示给用户。

　　最后，JobTracker会清理和回收该Job的相关资源，并通知TaskTracker进行相同的操作（比如删除中间结果文件）



## MapReduce处理流程

mapreduce 其实是分治算法的一种现，所谓分治算法就是“就是分而治之 ，将大的问题分解为相同类型的子问题（最好具有相同的规模），对子问题进行求解，然后合并成大问题的解。

　　mapreduce就是分治法的一种，将输入进行分片，然后交给不同的task进行处理，然后合并成最终的解。 
　　mapreduce实际的处理过程可以理解为**Input->Map->Sort->Combine->Partition->Reduce->Output**。

### Input阶段

　　　　数据以一定的格式传递给Mapper，有TextInputFormat，DBInputFormat，SequenceFileFormat等可以使用，在Job.setInputFormat可以设置，也可以自定义分片函数。

### map阶段

　　　　对输入的(key，value)进行处理，即map(k1,v1)->list(k2,v2),使用Job.setMapperClass进行设置。

### Sort阶段

　　　　对于Mapper的输出进行排序，使用Job.setOutputKeyComparatorClass进行设置，然后定义排序规则。

### Combine阶段

　　　　这个阶段对于Sort之后又相同key的结果进行合并，使用Job.setCombinerClass进行设置，也可以自定义Combine Class类。

### Partition阶段

　　　　将Mapper的中间结果按照key的范围划分为R份（Reduce作业的个数），默认使用HashPartioner（key.hashCode()&Integer.MAX_VALUE%numPartitions），也可以自定义划分的函数。

　　　　使用Job.setPartitionClass设置。

### Reduce阶段

　　　　对于Mapper阶段的结果进行进一步处理，Job.setReducerClass进行设置自定义的Reduce类。

### Output阶段

　　　　Reducer输出数据的格式。

