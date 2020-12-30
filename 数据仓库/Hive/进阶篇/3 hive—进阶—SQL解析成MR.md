[toc]
## SQL 解析原理

目前，Hive除了支持MapReduce计算引擎，还支持Spark和Tez这两中分布式计算引擎。

HDFS中最关键的一点就是，数据存储HDFS上是没有schema的概念的(schema:相当于表里面有列、字段、字段名称、字段与字段之间的分隔符等，这些就是schema信息)然而HDFS上的仅仅只是一个纯的文本文件而已，那么，没有schema，就没办法使用sql进行查询了啊，因此，在这种背景下，就有问题产生：如何为HDFS上的文件添加Schema信息？如果加上去，是否就可以通过SQL的方式进行处理了呢 于是强大的Hive出现了

前面我们说到了Hive 的SQL 实现不同于传统的数据库，因为SQL最终是翻译成MR 运行在Hadoop 集群上的，今天我们就学习一下是怎么翻译的，学习者一节需要你对MR有基本的了解，例如Mapper 和 Reducer 是干什么的，shuffle 是怎么一回事。

### SQL转化为MapReduce的过程

了解了SQL最终是翻译成MR 来执行的之后，。我们来看看Hive是如何将SQL转化为MapReduce任务的，整个编译过程分为六个阶段：

- Antlr定义SQL的语法规则，完成SQL词法，语法解析，将SQL转化为抽象语法树AST Tree
- 遍历AST Tree，抽象出查询的基本组成单元QueryBlock
- 遍历QueryBlock，**翻译为执行操作树OperatorTree**
- **逻辑层优化器进行OperatorTree变换**，合并不必要的ReduceSinkOperator，减少shuffle数据量
- 遍历OperatorTree，翻译为MapReduce任务
- 物理层优化器进行MapReduce任务的变换，生成最终的执行计划

### join 实现原理

我们这里说的是reduce-join(common join也叫做shuffle join)，在map阶段，map函数同时读取两个文件File1和File2，为了区分两种来源的key/value数据对，对每条数据打一个标签（tag）,比如：tag=0表示来自文件File1，tag=2表示来自文件File2。**即：map阶段的主要任务是对不同文件中的数据打标签**。然后将join 的关联字段作为key ，进行shuffle

在reduce阶段，reduce函数获取key相同的来自File1和File2文件的value list，然后对于同一个key，对File1和File2中的数据进行join（笛卡尔乘积）。**即：reduce阶段进行实际的连接操作**。

`select u.name, o.orderid from order o join user u on o.uid = u.uid;` 在map的输出value中为不同表的数据打上tag标记，在reduce阶段根据tag判断数据来源，然后根据SQL的select 顺序依次将需要的数据读取出来进行返回

![image-20201206211916602](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:19:17-image-20201206211916602.png)



### group by 实现原理

`select rank, isonline, count(*) from city group by rank, isonline;` 将GroupBy的字段组合为map的输出key值，利用MapReduce的排序，在reduce阶段保存LastKey(就是你组合起来的key)区分不同的key。

![image-20201206211942349](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:19:42-image-20201206211942349.png)

一般情况下我们分组都是为了统计，这个时候因为数据都放在了同一个reduce 上面，你就可以直接统计了，而不用担心数据分散在各个节点了

### distinct 实现原理

`select dealid, count(distinct uid) num from order group by dealid;` 当只有一个distinct字段时，只需要将GroupBy字段和Distinct字段组合为map输出key，利用mapreduce的排序，同时将GroupBy字段作为reduce的key，在reduce阶段保存LastKey即可完成去重

![image-20201206212008661](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/06/21:20:09-image-20201206212008661.png)





如果有多个distinct字段呢,`select dealid, count(distinct uid), count(distinct date) from order group by dealid;`

(1)如果仍然按照上面一个distinct字段的方法，即下图这种实现方式，无法跟据uid和date分别排序，也就无法通过LastKey去重，仍然需要在reduce阶段在内存中通过Hash去重

![2distinct-a.png](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/29/13:43:04-191719tk29ikd9dj9ggkrk.png)

(2)第二种实现方式，可以对所有的distinct字段编号，每行数据生成n行数据，那么相同字段就会分别排序，这时只需要在reduce阶段记录LastKey即可去重。

这种实现方式很好的利用了MapReduce的排序，节省了reduce阶段去重的内存消耗，但是缺点是增加了shuffle的数据量。需要注意的是，在生成reduce value时，除第一个distinct字段所在行需要保留value值，其余distinct数据行value字段均可为空。

![2distinct-b.png](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/12/29/13:43:55-191736yoa1qatjaez4u748.png)





## 总结

